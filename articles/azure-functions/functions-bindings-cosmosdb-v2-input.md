---
title: Enlace de entrada de Azure Cosmos DB para Functions 2.x y versiones superiores
description: Aprenda a usar el enlace de entrada de Azure Cosmos DB en Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.date: 02/24/2020
ms.author: cshoe
ms.custom: devx-track-csharp, devx-track-python
ms.openlocfilehash: 42932d712d6c4a94cad28aec924b88fbc126662b
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88212787"
---
# <a name="azure-cosmos-db-input-binding-for-azure-functions-2x-and-higher"></a>Enlace de entrada de Azure Cosmos DB para Azure Functions 2.x y versiones superiores

El enlace de entrada de Azure Cosmos DB usa SQL API para recuperar uno o varios documentos de Azure Cosmos DB y los pasa al parámetro de entrada de la función. Se puede determinar el identificador de documento o los parámetros de consulta según el desencadenador que invoca la función.

Para obtener información sobre los detalles de instalación y configuración, vea la [información general](./functions-bindings-cosmosdb-v2.md).

> [!NOTE]
> Si la colección tiene [particiones](../cosmos-db/partition-data.md#logical-partitions), las operaciones de búsqueda también tienen que especificar el valor de la clave de partición.
>

<a id="example" name="example"></a>

# <a name="c"></a>[C#](#tab/csharp)

En esta sección se incluyen los ejemplos siguientes:

* [Desencadenador de cola, buscar identificador de JSON](#queue-trigger-look-up-id-from-json-c)
* [Desencadenador de HTTP, buscar identificador de cadena de consulta](#http-trigger-look-up-id-from-query-string-c)
* [Desencadenador de HTTP, buscar identificador de datos de ruta](#http-trigger-look-up-id-from-route-data-c)
* [Desencadenador de HTTP, buscar identificador de datos de ruta, mediante SqlQuery](#http-trigger-look-up-id-from-route-data-using-sqlquery-c)
* [Desencadenador de HTTP, obtener varios documentos, mediante SqlQuery](#http-trigger-get-multiple-docs-using-sqlquery-c)
* [Desencadenador de HTTP, obtener varios documentos, mediante DocumentClient](#http-trigger-get-multiple-docs-using-documentclient-c)

Los ejemplos hacen referencia a un tipo de `ToDoItem` simple:

```cs
namespace CosmosDBSamplesV2
{
    public class ToDoItem
    {
        [JsonProperty("id")]
        public string Id { get; set; }
        
        [JsonProperty("partitionKey")]
        public string PartitionKey { get; set; }
        
        public string Description { get; set; }
    }
}
```

<a id="queue-trigger-look-up-id-from-json-c"></a>

### <a name="queue-trigger-look-up-id-from-json"></a>Desencadenador de cola, buscar identificador a partir de un JSON 

En el ejemplo siguiente se muestra una [función de C#](functions-dotnet-class-library.md) que recupera un documento individual. La función la desencadena un mensaje de la cola que contiene un objeto JSON. El desencadenador de la cola analiza el JSON en un objeto del tipo `ToDoItemLookup`, que contiene el identificador y el valor de la clave de partición que se van a buscar. El identificador y el valor de la clave de partición se utilizan para recuperar un documento `ToDoItem` de la base de datos y colección especificadas.

```cs
namespace CosmosDBSamplesV2
{
    public class ToDoItemLookup
    {
        public string ToDoItemId { get; set; }

        public string ToDoItemPartitionKeyValue { get; set; }
    }
}
```

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using Microsoft.Extensions.Logging;

namespace CosmosDBSamplesV2
{
    public static class DocByIdFromJSON
    {
        [FunctionName("DocByIdFromJSON")]
        public static void Run(
            [QueueTrigger("todoqueueforlookup")] ToDoItemLookup toDoItemLookup,
            [CosmosDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection",
                Id = "{ToDoItemId}",
                PartitionKey = "{ToDoItemPartitionKeyValue}")]ToDoItem toDoItem,
            ILogger log)
        {
            log.LogInformation($"C# Queue trigger function processed Id={toDoItemLookup?.ToDoItemId} Key={toDoItemLookup?.ToDoItemPartitionKeyValue}");

            if (toDoItem == null)
            {
                log.LogInformation($"ToDo item not found");
            }
            else
            {
                log.LogInformation($"Found ToDo item, Description={toDoItem.Description}");
            }
        }
    }
}
```

<a id="http-trigger-look-up-id-from-query-string-c"></a>

### <a name="http-trigger-look-up-id-from-query-string"></a>Desencadenador de HTTP, buscar identificador a partir de una cadena de consulta

En el ejemplo siguiente se muestra una [función de C#](functions-dotnet-class-library.md) que recupera un documento individual. La función la desencadena una solicitud HTTP que utiliza una cadena de consulta para especificar el identificador y el valor de la clave de partición que se van a buscar. El identificador y el valor de la clave de partición se utilizan para recuperar un documento `ToDoItem` de la base de datos y colección especificadas.

>[!NOTE]
>El parámetro de la cadena de consulta HTTP distingue mayúsculas de minúsculas.
>

```cs
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs.Host;
using Microsoft.Extensions.Logging;

namespace CosmosDBSamplesV2
{
    public static class DocByIdFromQueryString
    {
        [FunctionName("DocByIdFromQueryString")]
        public static IActionResult Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)]
                HttpRequest req,
            [CosmosDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection",
                Id = "{Query.id}",
                PartitionKey = "{Query.partitionKey}")] ToDoItem toDoItem,
            ILogger log)
        {
            log.LogInformation("C# HTTP trigger function processed a request.");

            if (toDoItem == null)
            {
                log.LogInformation($"ToDo item not found");
            }
            else
            {
                log.LogInformation($"Found ToDo item, Description={toDoItem.Description}");
            }
            return new OkResult();
        }
    }
}
```

<a id="http-trigger-look-up-id-from-route-data-c"></a>

### <a name="http-trigger-look-up-id-from-route-data"></a>Desencadenador de HTTP, buscar identificador a partir de datos de ruta

En el ejemplo siguiente se muestra una [función de C#](functions-dotnet-class-library.md) que recupera un documento individual. La función la desencadena una solicitud HTTP que utiliza datos de ruta para especificar el identificador y el valor de la clave de partición que se van a buscar. El identificador y el valor de la clave de partición se utilizan para recuperar un documento `ToDoItem` de la base de datos y colección especificadas.

```cs
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs.Host;
using Microsoft.Extensions.Logging;

namespace CosmosDBSamplesV2
{
    public static class DocByIdFromRouteData
    {
        [FunctionName("DocByIdFromRouteData")]
        public static IActionResult Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post",
                Route = "todoitems/{partitionKey}/{id}")]HttpRequest req,
            [CosmosDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection",
                Id = "{id}",
                PartitionKey = "{partitionKey}")] ToDoItem toDoItem,
            ILogger log)
        {
            log.LogInformation("C# HTTP trigger function processed a request.");

            if (toDoItem == null)
            {
                log.LogInformation($"ToDo item not found");
            }
            else
            {
                log.LogInformation($"Found ToDo item, Description={toDoItem.Description}");
            }
            return new OkResult();
        }
    }
}
```

<a id="http-trigger-look-up-id-from-route-data-using-sqlquery-c"></a>

### <a name="http-trigger-look-up-id-from-route-data-using-sqlquery"></a>Desencadenador de HTTP, buscar identificador a partir de datos de ruta mediante SqlQuery

En el ejemplo siguiente se muestra una [función de C#](functions-dotnet-class-library.md) que recupera un documento individual. La función la desencadena una solicitud HTTP que utiliza los datos de la ruta para especificar el identificador que se va a buscar. Dicho identificador se usa para recuperar un documento de `ToDoItem` de la base de datos y colección especificadas.

En el ejemplo se muestra cómo utilizar una expresión de enlace en el parámetro `SqlQuery`. Puede pasar datos de ruta al parámetro `SqlQuery` tal como se muestra, pero actualmente [no se pueden pasar valores de una cadena de consulta](https://github.com/Azure/azure-functions-host/issues/2554#issuecomment-392084583).

> [!NOTE]
> Si tiene que realizar una consulta solo con el identificador, se recomienda utilizar una búsqueda, como en los [ ejemplos anteriores](#http-trigger-look-up-id-from-query-string-c), ya que consumirá menos [unidades de solicitud](../cosmos-db/request-units.md). Las operaciones de lectura de punto (GET) son [más eficaces](../cosmos-db/optimize-cost-queries.md) que las consultas por identificador.
>

```cs
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs.Host;
using System.Collections.Generic;
using Microsoft.Extensions.Logging;

namespace CosmosDBSamplesV2
{
    public static class DocByIdFromRouteDataUsingSqlQuery
    {
        [FunctionName("DocByIdFromRouteDataUsingSqlQuery")]
        public static IActionResult Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post",
                Route = "todoitems2/{id}")]HttpRequest req,
            [CosmosDB("ToDoItems", "Items",
                ConnectionStringSetting = "CosmosDBConnection",
                SqlQuery = "select * from ToDoItems r where r.id = {id}")]
                IEnumerable<ToDoItem> toDoItems,
            ILogger log)
        {
            log.LogInformation("C# HTTP trigger function processed a request.");

            foreach (ToDoItem toDoItem in toDoItems)
            {
                log.LogInformation(toDoItem.Description);
            }
            return new OkResult();
        }
    }
}
```

<a id="http-trigger-get-multiple-docs-using-sqlquery-c"></a>

### <a name="http-trigger-get-multiple-docs-using-sqlquery"></a>Desencadenador de HTTP, obtener varios documentos mediante SqlQuery

En el ejemplo siguiente se muestra una [función de C#](functions-dotnet-class-library.md) que recupera una lista de documentos. La función la desencadena una solicitud HTTP. La consulta se especifica en la propiedad del atributo `SqlQuery`.

```cs
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs.Host;
using System.Collections.Generic;
using Microsoft.Extensions.Logging;

namespace CosmosDBSamplesV2
{
    public static class DocsBySqlQuery
    {
        [FunctionName("DocsBySqlQuery")]
        public static IActionResult Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)]
                HttpRequest req,
            [CosmosDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection",
                SqlQuery = "SELECT top 2 * FROM c order by c._ts desc")]
                IEnumerable<ToDoItem> toDoItems,
            ILogger log)
        {
            log.LogInformation("C# HTTP trigger function processed a request.");
            foreach (ToDoItem toDoItem in toDoItems)
            {
                log.LogInformation(toDoItem.Description);
            }
            return new OkResult();
        }
    }
}

```

<a id="http-trigger-get-multiple-docs-using-documentclient-c"></a>

### <a name="http-trigger-get-multiple-docs-using-documentclient"></a>Desencadenador de HTTP, obtener varios documentos mediante DocumentClient

En el ejemplo siguiente se muestra una [función de C#](functions-dotnet-class-library.md) que recupera una lista de documentos. La función la desencadena una solicitud HTTP. El código usa una instancia de `DocumentClient` que proporciona el enlace de Azure Cosmos DB para leer una lista de documentos. La instancia de `DocumentClient` también se puede usar para las operaciones de escritura.

> [!NOTE]
> También puede usar la interfaz [IDocumentClient](/dotnet/api/microsoft.azure.documents.idocumentclient?view=azure-dotnet) para simplificar las pruebas.

```cs
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.Documents.Client;
using Microsoft.Azure.Documents.Linq;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs.Host;
using Microsoft.Extensions.Logging;
using System;
using System.Linq;
using System.Threading.Tasks;

namespace CosmosDBSamplesV2
{
    public static class DocsByUsingDocumentClient
    {
        [FunctionName("DocsByUsingDocumentClient")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post",
                Route = null)]HttpRequest req,
            [CosmosDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection")] DocumentClient client,
            ILogger log)
        {
            log.LogInformation("C# HTTP trigger function processed a request.");

            var searchterm = req.Query["searchterm"];
            if (string.IsNullOrWhiteSpace(searchterm))
            {
                return (ActionResult)new NotFoundResult();
            }

            Uri collectionUri = UriFactory.CreateDocumentCollectionUri("ToDoItems", "Items");

            log.LogInformation($"Searching for: {searchterm}");

            IDocumentQuery<ToDoItem> query = client.CreateDocumentQuery<ToDoItem>(collectionUri)
                .Where(p => p.Description.Contains(searchterm))
                .AsDocumentQuery();

            while (query.HasMoreResults)
            {
                foreach (ToDoItem result in await query.ExecuteNextAsync())
                {
                    log.LogInformation(result.Description);
                }
            }
            return new OkResult();
        }
    }
}
```

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

En esta sección se incluyen los ejemplos siguientes:

* [Desencadenador de cola, buscar identificador de una cadena](#queue-trigger-look-up-id-from-string-c-script)
* [Desencadenador de colas, obtener varios documentos, mediante SqlQuery](#queue-trigger-get-multiple-docs-using-sqlquery-c-script)
* [Desencadenador de HTTP, buscar identificador de cadena de consulta](#http-trigger-look-up-id-from-query-string-c-script)
* [Desencadenador de HTTP, buscar identificador de datos de ruta](#http-trigger-look-up-id-from-route-data-c-script)
* [Desencadenador de HTTP, obtener varios documentos, mediante SqlQuery](#http-trigger-get-multiple-docs-using-sqlquery-c-script)
* [Desencadenador de HTTP, obtener varios documentos, mediante DocumentClient](#http-trigger-get-multiple-docs-using-documentclient-c-script)

Los ejemplos de desencadenador de HTTP hacen referencia a un tipo simple `ToDoItem`:

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

<a id="queue-trigger-look-up-id-from-string-c-script"></a>

### <a name="queue-trigger-look-up-id-from-string"></a>Desencadenador de cola, buscar identificador a partir de una cadena

En el ejemplo siguiente se muestra un enlace de entrada de Cosmos DB en un archivo *function.json* y una [función de script de C#](functions-reference-csharp.md) que usa el enlace. La función lee un solo documento y actualiza el valor de texto del documento.

Estos son los datos de enlace del archivo *function.json*:

```json
{
    "name": "inputDocument",
    "type": "cosmosDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "id" : "{queueTrigger}",
    "partitionKey": "{partition key value}",
    "connectionStringSetting": "MyAccount_COSMOSDB",
    "direction": "in"
}
```
En la sección de [configuración](#configuration) se explican estas propiedades.

Este es el código de script de C#:

```cs
    using System;

    // Change input document contents using Azure Cosmos DB input binding
    public static void Run(string myQueueItem, dynamic inputDocument)
    {
      inputDocument.text = "This has changed.";
    }
```

<a id="queue-trigger-get-multiple-docs-using-sqlquery-c-script"></a>

### <a name="queue-trigger-get-multiple-docs-using-sqlquery"></a>Desencadenador de colas, obtener varios documentos mediante SqlQuery

En el ejemplo siguiente se muestra un enlace de entrada de Azure Cosmos DB en un archivo *function.json* y una [función de script de C#](functions-reference-csharp.md) que usa el enlace. La función recupera varios documentos especificados por una consulta SQL con un desencadenador de cola para personalizar los parámetros de la consulta.

El desencadenador de cola proporciona un parámetro `departmentId`. Un mensaje de la cola de `{ "departmentId" : "Finance" }` devolvería todos los registros para el departamento de finanzas.

Estos son los datos de enlace del archivo *function.json*:

```json
{
    "name": "documents",
    "type": "cosmosDB",
    "direction": "in",
    "databaseName": "MyDb",
    "collectionName": "MyCollection",
    "sqlQuery": "SELECT * from c where c.departmentId = {departmentId}",
    "connectionStringSetting": "CosmosDBConnection"
}
```

En la sección de [configuración](#configuration) se explican estas propiedades.

Este es el código de script de C#:

```csharp
    public static void Run(QueuePayload myQueueItem, IEnumerable<dynamic> documents)
    {
        foreach (var doc in documents)
        {
            // operate on each document
        }
    }

    public class QueuePayload
    {
        public string departmentId { get; set; }
    }
```

<a id="http-trigger-look-up-id-from-query-string-c-script"></a>

### <a name="http-trigger-look-up-id-from-query-string"></a>Desencadenador de HTTP, buscar identificador a partir de una cadena de consulta

En el ejemplo siguiente se muestra una [función de script de C#](functions-reference-csharp.md) que recupera un documento individual. La función la desencadena una solicitud HTTP que utiliza una cadena de consulta para especificar el identificador y el valor de la clave de partición que se van a buscar. El identificador y el valor de la clave de partición se utilizan para recuperar un documento `ToDoItem` de la base de datos y colección especificadas.

Este es el archivo *function.json*:

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "type": "cosmosDB",
      "name": "toDoItem",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connectionStringSetting": "CosmosDBConnection",
      "direction": "in",
      "Id": "{Query.id}",
      "PartitionKey" : "{Query.partitionKeyValue}"
    }
  ],
  "disabled": false
}
```

Este es el código de script de C#:

```cs
using System.Net;
using Microsoft.Extensions.Logging;

public static HttpResponseMessage Run(HttpRequestMessage req, ToDoItem toDoItem, ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");

    if (toDoItem == null)
    {
         log.LogInformation($"ToDo item not found");
    }
    else
    {
        log.LogInformation($"Found ToDo item, Description={toDoItem.Description}");
    }
    return req.CreateResponse(HttpStatusCode.OK);
}
```

<a id="http-trigger-look-up-id-from-route-data-c-script"></a>

### <a name="http-trigger-look-up-id-from-route-data"></a>Desencadenador de HTTP, buscar identificador a partir de datos de ruta

En el ejemplo siguiente se muestra una [función de script de C#](functions-reference-csharp.md) que recupera un documento individual. La función la desencadena una solicitud HTTP que utiliza datos de ruta para especificar el identificador y el valor de la clave de partición que se van a buscar. El identificador y el valor de la clave de partición se utilizan para recuperar un documento `ToDoItem` de la base de datos y colección especificadas.

Este es el archivo *function.json*:

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": [
        "get",
        "post"
      ],
      "route":"todoitems/{partitionKeyValue}/{id}"
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "type": "cosmosDB",
      "name": "toDoItem",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connectionStringSetting": "CosmosDBConnection",
      "direction": "in",
      "Id": "{id}",
      "PartitionKey": "{partitionKeyValue}"
    }
  ],
  "disabled": false
}
```

Este es el código de script de C#:

```cs
using System.Net;
using Microsoft.Extensions.Logging;

public static HttpResponseMessage Run(HttpRequestMessage req, ToDoItem toDoItem, ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");

    if (toDoItem == null)
    {
         log.LogInformation($"ToDo item not found");
    }
    else
    {
        log.LogInformation($"Found ToDo item, Description={toDoItem.Description}");
    }
    return req.CreateResponse(HttpStatusCode.OK);
}
```

<a id="http-trigger-get-multiple-docs-using-sqlquery-c-script"></a>

### <a name="http-trigger-get-multiple-docs-using-sqlquery"></a>Desencadenador de HTTP, obtener varios documentos mediante SqlQuery

En el ejemplo siguiente se muestra una [función de script de C#](functions-reference-csharp.md) que recupera una lista de documentos. La función la desencadena una solicitud HTTP. La consulta se especifica en la propiedad del atributo `SqlQuery`.

Este es el archivo *function.json*:

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "type": "cosmosDB",
      "name": "toDoItems",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connectionStringSetting": "CosmosDBConnection",
      "direction": "in",
      "sqlQuery": "SELECT top 2 * FROM c order by c._ts desc"
    }
  ],
  "disabled": false
}
```

Este es el código de script de C#:

```cs
using System.Net;
using Microsoft.Extensions.Logging;

public static HttpResponseMessage Run(HttpRequestMessage req, IEnumerable<ToDoItem> toDoItems, ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");

    foreach (ToDoItem toDoItem in toDoItems)
    {
        log.LogInformation(toDoItem.Description);
    }
    return req.CreateResponse(HttpStatusCode.OK);
}
```

<a id="http-trigger-get-multiple-docs-using-documentclient-c-script"></a>

### <a name="http-trigger-get-multiple-docs-using-documentclient"></a>Desencadenador de HTTP, obtener varios documentos mediante DocumentClient

En el ejemplo siguiente se muestra una [función de script de C#](functions-reference-csharp.md) que recupera una lista de documentos. La función la desencadena una solicitud HTTP. El código usa una instancia de `DocumentClient` que proporciona el enlace de Azure Cosmos DB para leer una lista de documentos. La instancia de `DocumentClient` también se puede usar para las operaciones de escritura.

Este es el archivo *function.json*:

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "type": "cosmosDB",
      "name": "client",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connectionStringSetting": "CosmosDBConnection",
      "direction": "inout"
    }
  ],
  "disabled": false
}
```

Este es el código de script de C#:

```cs
#r "Microsoft.Azure.Documents.Client"

using System.Net;
using Microsoft.Azure.Documents.Client;
using Microsoft.Azure.Documents.Linq;
using Microsoft.Extensions.Logging;

public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, DocumentClient client, ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");

    Uri collectionUri = UriFactory.CreateDocumentCollectionUri("ToDoItems", "Items");
    string searchterm = req.GetQueryNameValuePairs()
        .FirstOrDefault(q => string.Compare(q.Key, "searchterm", true) == 0)
        .Value;

    if (searchterm == null)
    {
        return req.CreateResponse(HttpStatusCode.NotFound);
    }

    log.LogInformation($"Searching for word: {searchterm} using Uri: {collectionUri.ToString()}");
    IDocumentQuery<ToDoItem> query = client.CreateDocumentQuery<ToDoItem>(collectionUri)
        .Where(p => p.Description.Contains(searchterm))
        .AsDocumentQuery();

    while (query.HasMoreResults)
    {
        foreach (ToDoItem result in await query.ExecuteNextAsync())
        {
            log.LogInformation(result.Description);
        }
    }
    return req.CreateResponse(HttpStatusCode.OK);
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Esta sección contiene los siguientes ejemplos que leen un solo documento mediante la especificación de un valor de identificación desde varios orígenes:

* [Desencadenador de cola, buscar identificador de JSON](#queue-trigger-look-up-id-from-json-javascript)
* [Desencadenador de HTTP, buscar identificador de cadena de consulta](#http-trigger-look-up-id-from-query-string-javascript)
* [Desencadenador de HTTP, buscar identificador de datos de ruta](#http-trigger-look-up-id-from-route-data-javascript)
* [Desencadenador de colas, obtener varios documentos, mediante SqlQuery](#queue-trigger-get-multiple-docs-using-sqlquery-javascript)

<a id="queue-trigger-look-up-id-from-json-javascript"></a>

### <a name="queue-trigger-look-up-id-from-json"></a>Desencadenador de cola, buscar identificador a partir de un JSON

En el ejemplo siguiente se muestra un enlace de entrada de Cosmos DB en un archivo *function.json* y una [función de JavaScript](functions-reference-node.md) que usa el enlace. La función lee un solo documento y actualiza el valor de texto del documento.

Estos son los datos de enlace del archivo *function.json*:

```json
{
    "name": "inputDocumentIn",
    "type": "cosmosDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "id" : "{queueTrigger_payload_property}",
    "partitionKey": "{queueTrigger_payload_property}",
    "connectionStringSetting": "MyAccount_COSMOSDB",
    "direction": "in"
},
{
    "name": "inputDocumentOut",
    "type": "cosmosDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "createIfNotExists": false,
    "partitionKey": "{queueTrigger_payload_property}",
    "connectionStringSetting": "MyAccount_COSMOSDB",
    "direction": "out"
}
```

En la sección de [configuración](#configuration) se explican estas propiedades.

Este es el código de JavaScript:

```javascript
    // Change input document contents using Azure Cosmos DB input binding, using context.bindings.inputDocumentOut
    module.exports = function (context) {
    context.bindings.inputDocumentOut = context.bindings.inputDocumentIn;
    context.bindings.inputDocumentOut.text = "This was updated!";
    context.done();
    };
```

<a id="http-trigger-look-up-id-from-query-string-javascript"></a>

### <a name="http-trigger-look-up-id-from-query-string"></a>Desencadenador de HTTP, buscar identificador a partir de una cadena de consulta

En el ejemplo siguiente se muestra una [función de JavaScript](functions-reference-node.md) que recupera un documento individual. La función la desencadena una solicitud HTTP que utiliza una cadena de consulta para especificar el identificador y el valor de la clave de partición que se van a buscar. El identificador y el valor de la clave de partición se utilizan para recuperar un documento `ToDoItem` de la base de datos y colección especificadas.

Este es el archivo *function.json*:

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "type": "cosmosDB",
      "name": "toDoItem",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connectionStringSetting": "CosmosDBConnection",
      "direction": "in",
      "Id": "{Query.id}",
      "PartitionKey": "{Query.partitionKeyValue}"
    }
  ],
  "disabled": false
}
```

Este es el código de JavaScript:

```javascript
module.exports = function (context, req, toDoItem) {
    context.log('JavaScript queue trigger function processed work item');
    if (!toDoItem)
    {
        context.log("ToDo item not found");
    }
    else
    {
        context.log("Found ToDo item, Description=" + toDoItem.Description);
    }

    context.done();
};
```

<a id="http-trigger-look-up-id-from-route-data-javascript"></a>

### <a name="http-trigger-look-up-id-from-route-data"></a>Desencadenador de HTTP, buscar identificador a partir de datos de ruta

En el ejemplo siguiente se muestra una [función de JavaScript](functions-reference-node.md) que recupera un documento individual. La función la desencadena una solicitud HTTP que utiliza datos de ruta para especificar el identificador y el valor de la clave de partición que se van a buscar. El identificador y el valor de la clave de partición se utilizan para recuperar un documento `ToDoItem` de la base de datos y colección especificadas.

Este es el archivo *function.json*:

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": [
        "get",
        "post"
      ],
      "route":"todoitems/{partitionKeyValue}/{id}"
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "type": "cosmosDB",
      "name": "toDoItem",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connectionStringSetting": "CosmosDBConnection",
      "direction": "in",
      "Id": "{id}",
      "PartitionKey": "{partitionKeyValue}"
    }
  ],
  "disabled": false
}
```

Este es el código de JavaScript:

```javascript
module.exports = function (context, req, toDoItem) {
    context.log('JavaScript queue trigger function processed work item');
    if (!toDoItem)
    {
        context.log("ToDo item not found");
    }
    else
    {
        context.log("Found ToDo item, Description=" + toDoItem.Description);
    }

    context.done();
};
```

<a id="queue-trigger-get-multiple-docs-using-sqlquery-javascript"></a>

### <a name="queue-trigger-get-multiple-docs-using-sqlquery"></a>Desencadenador de colas, obtener varios documentos mediante SqlQuery

En el ejemplo siguiente se muestra un enlace de entrada de Azure Cosmos DB en un archivo *function.json* y una [función de JavaScript](functions-reference-node.md) que usa el enlace. La función recupera varios documentos especificados por una consulta SQL con un desencadenador de cola para personalizar los parámetros de la consulta.

El desencadenador de cola proporciona un parámetro `departmentId`. Un mensaje de la cola de `{ "departmentId" : "Finance" }` devolvería todos los registros para el departamento de finanzas.

Estos son los datos de enlace del archivo *function.json*:

```json
{
    "name": "documents",
    "type": "cosmosDB",
    "direction": "in",
    "databaseName": "MyDb",
    "collectionName": "MyCollection",
    "sqlQuery": "SELECT * from c where c.departmentId = {departmentId}",
    "connectionStringSetting": "CosmosDBConnection"
}
```

En la sección de [configuración](#configuration) se explican estas propiedades.

Este es el código de JavaScript:

```javascript
    module.exports = function (context, input) {
        var documents = context.bindings.documents;
        for (var i = 0; i < documents.length; i++) {
            var document = documents[i];
            // operate on each document
        }
        context.done();
    };
```

# <a name="python"></a>[Python](#tab/python)

Esta sección contiene los siguientes ejemplos que leen un solo documento mediante la especificación de un valor de identificación desde varios orígenes:

* [Desencadenador de cola, buscar identificador de JSON](#queue-trigger-look-up-id-from-json-python)
* [Desencadenador de HTTP, buscar identificador de cadena de consulta](#http-trigger-look-up-id-from-query-string-python)
* [Desencadenador de HTTP, buscar identificador de datos de ruta](#http-trigger-look-up-id-from-route-data-python)
* [Desencadenador de colas, obtener varios documentos, mediante SqlQuery](#queue-trigger-get-multiple-docs-using-sqlquery-python)

<a id="queue-trigger-look-up-id-from-json-python"></a>

### <a name="queue-trigger-look-up-id-from-json"></a>Desencadenador de cola, buscar identificador a partir de un JSON

En el ejemplo siguiente se muestra un enlace de entrada de Cosmos DB en un archivo *function.json* y una [función de Python](functions-reference-python.md) que usa el enlace. La función lee un solo documento y actualiza el valor de texto del documento.

Estos son los datos de enlace del archivo *function.json*:

```json
{
    "name": "documents",
    "type": "cosmosDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "id" : "{queueTrigger_payload_property}",
    "partitionKey": "{queueTrigger_payload_property}",
    "connectionStringSetting": "MyAccount_COSMOSDB",
    "direction": "in"
},
{
    "name": "$return",
    "type": "cosmosDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "createIfNotExists": false,
    "partitionKey": "{queueTrigger_payload_property}",
    "connectionStringSetting": "MyAccount_COSMOSDB",
    "direction": "out"
}
```

En la sección de [configuración](#configuration) se explican estas propiedades.

Este es el código de Python:

```python
import azure.functions as func


def main(queuemsg: func.QueueMessage, documents: func.DocumentList) -> func.Document:
    if documents:
        document = documents[0]
        document['text'] = 'This was updated!'
        return document
```

<a id="http-trigger-look-up-id-from-query-string-python"></a>

### <a name="http-trigger-look-up-id-from-query-string"></a>Desencadenador de HTTP, buscar identificador a partir de una cadena de consulta

En el ejemplo siguiente se muestra una [función de Python](functions-reference-python.md) que recupera un documento individual. La función la desencadena una solicitud HTTP que utiliza una cadena de consulta para especificar el identificador y el valor de la clave de partición que se van a buscar. El identificador y el valor de la clave de partición se utilizan para recuperar un documento `ToDoItem` de la base de datos y colección especificadas.

Este es el archivo *function.json*:

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "type": "cosmosDB",
      "name": "todoitems",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connectionStringSetting": "CosmosDBConnection",
      "direction": "in",
      "Id": "{Query.id}",
      "PartitionKey": "{Query.partitionKeyValue}"
    }
  ],
  "scriptFile": "__init__.py"
}
```

Este es el código de Python:

```python
import logging
import azure.functions as func


def main(req: func.HttpRequest, todoitems: func.DocumentList) -> str:
    if not todoitems:
        logging.warning("ToDo item not found")
    else:
        logging.info("Found ToDo item, Description=%s",
                     todoitems[0]['description'])

    return 'OK'
```

<a id="http-trigger-look-up-id-from-route-data-python"></a>

### <a name="http-trigger-look-up-id-from-route-data"></a>Desencadenador de HTTP, buscar identificador a partir de datos de ruta

En el ejemplo siguiente se muestra una [función de Python](functions-reference-python.md) que recupera un documento individual. La función la desencadena una solicitud HTTP que utiliza datos de ruta para especificar el identificador y el valor de la clave de partición que se van a buscar. El identificador y el valor de la clave de partición se utilizan para recuperar un documento `ToDoItem` de la base de datos y colección especificadas.

Este es el archivo *function.json*:

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": [
        "get",
        "post"
      ],
      "route":"todoitems/{partitionKeyValue}/{id}"
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "type": "cosmosDB",
      "name": "todoitems",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connection": "CosmosDBConnection",
      "direction": "in",
      "Id": "{id}",
      "PartitionKey": "{partitionKeyValue}"
    }
  ],
  "disabled": false,
  "scriptFile": "__init__.py"
}
```

Este es el código de Python:

```python
import logging
import azure.functions as func


def main(req: func.HttpRequest, todoitems: func.DocumentList) -> str:
    if not todoitems:
        logging.warning("ToDo item not found")
    else:
        logging.info("Found ToDo item, Description=%s",
                     todoitems[0]['description'])
    return 'OK'
```

<a id="queue-trigger-get-multiple-docs-using-sqlquery-python"></a>

### <a name="queue-trigger-get-multiple-docs-using-sqlquery"></a>Desencadenador de colas, obtener varios documentos mediante SqlQuery

En el ejemplo siguiente se muestra un enlace de entrada de Azure Cosmos DB en un archivo *function.json* y una [función de Python](functions-reference-python.md) que usa dicho enlace. La función recupera varios documentos especificados por una consulta SQL con un desencadenador de cola para personalizar los parámetros de la consulta.

El desencadenador de cola proporciona un parámetro `departmentId`. Un mensaje de la cola de `{ "departmentId" : "Finance" }` devolvería todos los registros para el departamento de finanzas.

Estos son los datos de enlace del archivo *function.json*:

```json
{
    "name": "documents",
    "type": "cosmosDB",
    "direction": "in",
    "databaseName": "MyDb",
    "collectionName": "MyCollection",
    "sqlQuery": "SELECT * from c where c.departmentId = {departmentId}",
    "connectionStringSetting": "CosmosDBConnection"
}
```

En la sección de [configuración](#configuration) se explican estas propiedades.

Este es el código de Python:

```python
import azure.functions as func

def main(queuemsg: func.QueueMessage, documents: func.DocumentList):
    for document in documents:
        # operate on each document
```

# <a name="java"></a>[Java](#tab/java)

En esta sección se incluyen los ejemplos siguientes:

* [Desencadenador de HTTP, buscar identificador de cadena de consulta - Parámetro String](#http-trigger-look-up-id-from-query-string---string-parameter-java)
* [Desencadenador de HTTP, buscar identificador de cadena de consulta - Parámetro POJO](#http-trigger-look-up-id-from-query-string---pojo-parameter-java)
* [Desencadenador de HTTP, buscar identificador de datos de ruta](#http-trigger-look-up-id-from-route-data-java)
* [Desencadenador de HTTP, buscar identificador de datos de ruta, mediante SqlQuery](#http-trigger-look-up-id-from-route-data-using-sqlquery-java)
* [Desencadenador de HTTP, obtener varios documentos de datos de ruta, mediante SqlQuery](#http-trigger-get-multiple-docs-from-route-data-using-sqlquery-java)

Los ejemplos hacen referencia a un tipo de `ToDoItem` simple:

```java
public class ToDoItem {

  private String id;
  private String description;

  public String getId() {
    return id;
  }

  public String getDescription() {
    return description;
  }

  @Override
  public String toString() {
    return "ToDoItem={id=" + id + ",description=" + description + "}";
  }
}
```

<a id="http-trigger-look-up-id-from-query-string---string-parameter-java"></a>

### <a name="http-trigger-look-up-id-from-query-string---string-parameter"></a>Desencadenador HTTP, buscar identificador en la cadena de consulta: parámetro de cadena

En el ejemplo siguiente se muestra una función de Java que recupera un documento individual. La función la desencadena una solicitud HTTP que utiliza una cadena de consulta para especificar el identificador y el valor de la clave de partición que se van a buscar. El identificador y el valor de la clave de partición se utilizan para recuperar un documento de la base de datos y colección especificadas, en forma de cadena.

```java
public class DocByIdFromQueryString {

    @FunctionName("DocByIdFromQueryString")
    public HttpResponseMessage run(
            @HttpTrigger(name = "req",
              methods = {HttpMethod.GET, HttpMethod.POST},
              authLevel = AuthorizationLevel.ANONYMOUS)
            HttpRequestMessage<Optional<String>> request,
            @CosmosDBInput(name = "database",
              databaseName = "ToDoList",
              collectionName = "Items",
              id = "{Query.id}",
              partitionKey = "{Query.partitionKeyValue}",
              connectionStringSetting = "Cosmos_DB_Connection_String")
            Optional<String> item,
            final ExecutionContext context) {

        // Item list
        context.getLogger().info("Parameters are: " + request.getQueryParameters());
        context.getLogger().info("String from the database is " + (item.isPresent() ? item.get() : null));

        // Convert and display
        if (!item.isPresent()) {
            return request.createResponseBuilder(HttpStatus.BAD_REQUEST)
                          .body("Document not found.")
                          .build();
        }
        else {
            // return JSON from Cosmos. Alternatively, we can parse the JSON string
            // and return an enriched JSON object.
            return request.createResponseBuilder(HttpStatus.OK)
                          .header("Content-Type", "application/json")
                          .body(item.get())
                          .build();
        }
    }
}
 ```

En la [biblioteca en tiempo de ejecución de funciones de Java](/java/api/overview/azure/functions/runtime), utilice la anotación `@CosmosDBInput` en los parámetros de la función cuyo valor provendría de Cosmos DB.  Esta anotación se puede usar con tipos nativos de Java, POJO o valores que aceptan valores NULL mediante `Optional<T>`.

<a id="http-trigger-look-up-id-from-query-string---pojo-parameter-java"></a>

### <a name="http-trigger-look-up-id-from-query-string---pojo-parameter"></a>Desencadenador HTTP, buscar identificador en la cadena de consulta: parámetro POJO

En el ejemplo siguiente se muestra una función de Java que recupera un documento individual. La función la desencadena una solicitud HTTP que utiliza una cadena de consulta para especificar el identificador y el valor de la clave de partición que se van a buscar. El identificador y el valor de la clave de partición se utilizan para recuperar un documento de la base de datos y colección especificadas. A continuación, el documento se convierte en una instancia de la anotación ```ToDoItem``` POJO creada previamente y se pasa como argumento a la función.

```java
public class DocByIdFromQueryStringPojo {

    @FunctionName("DocByIdFromQueryStringPojo")
    public HttpResponseMessage run(
            @HttpTrigger(name = "req",
              methods = {HttpMethod.GET, HttpMethod.POST},
              authLevel = AuthorizationLevel.ANONYMOUS)
            HttpRequestMessage<Optional<String>> request,
            @CosmosDBInput(name = "database",
              databaseName = "ToDoList",
              collectionName = "Items",
              id = "{Query.id}",
              partitionKey = "{Query.partitionKeyValue}",
              connectionStringSetting = "Cosmos_DB_Connection_String")
            ToDoItem item,
            final ExecutionContext context) {

        // Item list
        context.getLogger().info("Parameters are: " + request.getQueryParameters());
        context.getLogger().info("Item from the database is " + item);

        // Convert and display
        if (item == null) {
            return request.createResponseBuilder(HttpStatus.BAD_REQUEST)
                          .body("Document not found.")
                          .build();
        }
        else {
            return request.createResponseBuilder(HttpStatus.OK)
                          .header("Content-Type", "application/json")
                          .body(item)
                          .build();
        }
    }
}
 ```

<a id="http-trigger-look-up-id-from-route-data-java"></a>

### <a name="http-trigger-look-up-id-from-route-data"></a>Desencadenador de HTTP, buscar identificador a partir de datos de ruta

En el ejemplo siguiente se muestra una función de Java que recupera un documento individual. La función la desencadena una solicitud HTTP que utiliza un parámetro de ruta para especificar el identificador y el valor de la clave de partición que se van a buscar. El identificador y el valor de la clave de partición se utilizan para recuperar un documento de la base de datos y colección especificadas, que se devuelve como un objeto ```Optional<String>```.

```java
public class DocByIdFromRoute {

    @FunctionName("DocByIdFromRoute")
    public HttpResponseMessage run(
            @HttpTrigger(name = "req",
              methods = {HttpMethod.GET, HttpMethod.POST},
              authLevel = AuthorizationLevel.ANONYMOUS,
              route = "todoitems/{partitionKeyValue}/{id}")
            HttpRequestMessage<Optional<String>> request,
            @CosmosDBInput(name = "database",
              databaseName = "ToDoList",
              collectionName = "Items",
              id = "{id}",
              partitionKey = "{partitionKeyValue}",
              connectionStringSetting = "Cosmos_DB_Connection_String")
            Optional<String> item,
            final ExecutionContext context) {

        // Item list
        context.getLogger().info("Parameters are: " + request.getQueryParameters());
        context.getLogger().info("String from the database is " + (item.isPresent() ? item.get() : null));

        // Convert and display
        if (!item.isPresent()) {
            return request.createResponseBuilder(HttpStatus.BAD_REQUEST)
                          .body("Document not found.")
                          .build();
        }
        else {
            // return JSON from Cosmos. Alternatively, we can parse the JSON string
            // and return an enriched JSON object.
            return request.createResponseBuilder(HttpStatus.OK)
                          .header("Content-Type", "application/json")
                          .body(item.get())
                          .build();
        }
    }
}
 ```

 <a id="http-trigger-look-up-id-from-route-data-using-sqlquery-java"></a>

### <a name="http-trigger-look-up-id-from-route-data-using-sqlquery"></a>Desencadenador de HTTP, buscar identificador a partir de datos de ruta mediante SqlQuery

En el ejemplo siguiente se muestra una función de Java que recupera un documento individual. La función la desencadena una solicitud HTTP que utiliza un parámetro de ruta para especificar el identificador que se va a buscar. Ese identificador se usa para recuperar un documento de la base de datos y la colección especificadas, convirtiendo el conjunto de resultados en una anotación ```ToDoItem[]```, ya que pueden devolverse muchos documentos, en función de los criterios de consulta.

> [!NOTE]
> Si tiene que realizar una consulta solo con el identificador, se recomienda utilizar una búsqueda, como en los [ ejemplos anteriores](#http-trigger-look-up-id-from-query-string---pojo-parameter-java), ya que consumirá menos [unidades de solicitud](../cosmos-db/request-units.md). Las operaciones de lectura de punto (GET) son [más eficaces](../cosmos-db/optimize-cost-queries.md) que las consultas por identificador.
>

```java
public class DocByIdFromRouteSqlQuery {

    @FunctionName("DocByIdFromRouteSqlQuery")
    public HttpResponseMessage run(
            @HttpTrigger(name = "req",
              methods = {HttpMethod.GET, HttpMethod.POST},
              authLevel = AuthorizationLevel.ANONYMOUS,
              route = "todoitems2/{id}")
            HttpRequestMessage<Optional<String>> request,
            @CosmosDBInput(name = "database",
              databaseName = "ToDoList",
              collectionName = "Items",
              sqlQuery = "select * from Items r where r.id = {id}",
              connectionStringSetting = "Cosmos_DB_Connection_String")
            ToDoItem[] item,
            final ExecutionContext context) {

        // Item list
        context.getLogger().info("Parameters are: " + request.getQueryParameters());
        context.getLogger().info("Items from the database are " + item);

        // Convert and display
        if (item == null) {
            return request.createResponseBuilder(HttpStatus.BAD_REQUEST)
                          .body("Document not found.")
                          .build();
        }
        else {
            return request.createResponseBuilder(HttpStatus.OK)
                          .header("Content-Type", "application/json")
                          .body(item)
                          .build();
        }
    }
}
 ```

 <a id="http-trigger-get-multiple-docs-from-route-data-using-sqlquery-java"></a>

### <a name="http-trigger-get-multiple-docs-from-route-data-using-sqlquery"></a>Desencadenador HTTP, obtener varios documentos de los datos de ruta, mediante SqlQuery

En el ejemplo siguiente se muestra una función de Java que recupera varios documentos. La función la desencadena una solicitud HTTP que utiliza un parámetro de ruta ```desc``` para especificar la cadena que se va a buscar en el campo ```description```. El término de búsqueda se usa para recuperar una colección de documentos de la base de datos y de la colección especificadas, lo que convierte el conjunto de resultados en una anotación ```ToDoItem[]``` y la pasa como un argumento a la función.

```java
public class DocsFromRouteSqlQuery {

    @FunctionName("DocsFromRouteSqlQuery")
    public HttpResponseMessage run(
            @HttpTrigger(name = "req",
              methods = {HttpMethod.GET},
              authLevel = AuthorizationLevel.ANONYMOUS,
              route = "todoitems3/{desc}")
            HttpRequestMessage<Optional<String>> request,
            @CosmosDBInput(name = "database",
              databaseName = "ToDoList",
              collectionName = "Items",
              sqlQuery = "select * from Items r where contains(r.description, {desc})",
              connectionStringSetting = "Cosmos_DB_Connection_String")
            ToDoItem[] items,
            final ExecutionContext context) {

        // Item list
        context.getLogger().info("Parameters are: " + request.getQueryParameters());
        context.getLogger().info("Number of items from the database is " + (items == null ? 0 : items.length));

        // Convert and display
        if (items == null) {
            return request.createResponseBuilder(HttpStatus.BAD_REQUEST)
                          .body("No documents found.")
                          .build();
        }
        else {
            return request.createResponseBuilder(HttpStatus.OK)
                          .header("Content-Type", "application/json")
                          .body(items)
                          .build();
        }
    }
}
 ```

 ---

## <a name="attributes-and-annotations"></a>Atributos y anotaciones

# <a name="c"></a>[C#](#tab/csharp)

En las [bibliotecas de clases de C#](functions-dotnet-class-library.md), use el atributo [CosmosDB](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.CosmosDB/CosmosDBAttribute.cs).

El constructor del atributo toma el nombre de la base de datos y el nombre de la colección. Para información sobre esos valores y otras propiedades que puede configurar, consulte [la sección de configuración siguiente](#configuration).

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

El script de C# no admite atributos.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

JavaScript no admite atributos.

# <a name="python"></a>[Python](#tab/python)

Python no admite atributos.

# <a name="java"></a>[Java](#tab/java)

En la [biblioteca en tiempo de ejecución de funciones de Java](/java/api/overview/azure/functions/runtime), utilice la anotación `@CosmosDBOutput` en los parámetros que escriben en Cosmos DB. El tipo de parámetro de anotación debe ser `OutputBinding<T>`, donde `T` es un tipo nativo de Java o un POJO.

---

## <a name="configuration"></a>Configuración

En la siguiente tabla se explican las propiedades de configuración de enlace que se definen en el archivo *function.json* y el atributo `CosmosDB`.

|Propiedad de function.json | Propiedad de atributo |Descripción|
|---------|---------|----------------------|
|**type**     | N/D | Se debe establecer en `cosmosDB`.        |
|**direction**     | N/D | Se debe establecer en `in`.         |
|**name**     | N/D | Nombre del parámetro de enlace que representa al documento en la función.  |
|**databaseName** |**DatabaseName** |Base de datos que contiene el documento.        |
|**collectionName** |**CollectionName** | Nombre de la colección que contiene el documento. |
|**id**    | **Id** | Identificador del documento que se va a recuperar. Esta propiedad es compatible con [expresiones de enlace](./functions-bindings-expressions-patterns.md). No establezca las propiedades `id` y **sqlQuery** a la vez. Si no establece alguna de ellas, se recupera toda la colección. |
|**sqlQuery**  |**SqlQuery**  | Consulta SQL de Azure Cosmos DB que se usa para recuperar varios documentos. La propiedad es compatible con los enlaces en tiempo de ejecución, como en este ejemplo: `SELECT * FROM c where c.departmentId = {departmentId}`. No establezca las propiedades `id` y `sqlQuery` a la vez. Si no establece alguna de ellas, se recupera toda la colección.|
|**connectionStringSetting**     |**ConnectionStringSetting**|Nombre de la configuración de aplicación que contiene la cadena de conexión de Azure Cosmos DB. |
|**partitionKey**|**PartitionKey**|Especifica el valor de la clave de partición para la búsqueda. Puede incluir parámetros de enlace. Se requiere para las búsquedas en colecciones [con particiones](../cosmos-db/partition-data.md#logical-partitions).|
|**preferredLocations**| **PreferredLocations**| (Opcional) Defina las ubicaciones preferidas (regiones) para las cuentas de base de datos con replicación geográfica en el servicio de Azure Cosmos DB. Los valores deben estar separados por comas. Por ejemplo, "Este de EE. UU., Centro-sur de EE. UU. Norte de Europa". |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>Uso

# <a name="c"></a>[C#](#tab/csharp)

Cuando se sale de la función correctamente, los cambios realizados en el documento de entrada mediante parámetros de entrada con nombre se guardan automáticamente.

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

Cuando se sale de la función correctamente, los cambios realizados en el documento de entrada mediante parámetros de entrada con nombre se guardan automáticamente.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Las actualizaciones no se realizan automáticamente al cerrar la función. Por el contrario, use `context.bindings.<documentName>In` y `context.bindings.<documentName>Out` para realizar las actualizaciones. Consulte el ejemplo de JavaScript.

# <a name="python"></a>[Python](#tab/python)

Los datos se ponen a disposición de la función mediante un parámetro `DocumentList`. Los cambios realizados en el documento no se conservan automáticamente.

# <a name="java"></a>[Java](#tab/java)

En la [biblioteca en tiempo de ejecución de funciones de Java](/java/api/overview/azure/functions/runtime), la anotación [@CosmosDBInput](/java/api/com.microsoft.azure.functions.annotation.cosmosdbinput) expone los datos de Cosmos DB a la función. Esta anotación se puede usar con tipos nativos de Java, POJO o valores que aceptan valores NULL mediante `Optional<T>`.

---

## <a name="next-steps"></a>Pasos siguientes

- [Ejecución de una función cuando se crea o modifica un documento de Azure Cosmos DB (desencadenador)](./functions-bindings-cosmosdb-v2-trigger.md)
- [Guardar los cambios en un documento de Azure Cosmos DB (enlace de salida)](./functions-bindings-cosmosdb-v2-output.md)
