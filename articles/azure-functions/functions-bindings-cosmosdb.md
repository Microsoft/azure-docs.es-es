---
title: Enlaces de Azure Cosmos DB para Functions 1.x
description: Descubra cómo utilizar desencadenadores y enlaces de almacenamiento de Azure Cosmos DB en Azure Functions.
author: craigshoemaker
ms.author: cshoe
ms.topic: reference
ms.date: 11/21/2017
ms.custom: seodec18
ms.openlocfilehash: 37c98ca2fbffddd064b55192e9b5d83c948d88e6
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87041959"
---
# <a name="azure-cosmos-db-bindings-for-azure-functions-1x"></a>Enlaces de Azure Cosmos DB para Azure Functions 1.x

> [!div class="op_single_selector" title1="Seleccione la versión del entorno de ejecución de Azure Functions que usa: "]
> * [Versión 1](functions-bindings-cosmosdb.md)
> * [Versión 2](functions-bindings-cosmosdb-v2.md)

En este artículo se explica cómo trabajar con enlaces de [Azure Cosmos DB](../cosmos-db/serverless-computing-database.md) en Azure Functions. Azure Functions enlaces de desencadenador, de entrada y de salida para Azure Cosmos DB.

> [!NOTE]
> Este artículo trata sobre Azure Functions 1.x. Para más información acerca del uso de estos enlaces en Functions 2.x y versiones posteriores, consulte [Enlaces de Azure Cosmos DB para Azure Functions 2.x](functions-bindings-cosmosdb-v2.md).
>
>Este enlace se llamaba originalmente DocumentDB. En la versión 1.x de Functions, solo se cambió el nombre del desencadenador a Cosmos DB; el enlace de entrada, el enlace de salida y el paquete NuGet conservan el nombre DocumentDB.

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

> [!NOTE]
> Solo se admiten enlaces de Azure Cosmos DB para su uso con la API de SQL. Para todas las demás API de Azure Cosmos DB, debe acceder a la base de datos de la función mediante el cliente estático de la API, lo que incluye la [API de Azure Cosmos DB para MongoDB](../cosmos-db/mongodb-introduction.md), [Cassandra API](../cosmos-db/cassandra-introduction.md), [Gremlin API](../cosmos-db/graph-introduction.md) y [Table API](../cosmos-db/table-introduction.md).

## <a name="packages---functions-1x"></a>Paquetes: Functions 1.x

Los enlaces de Azure Cosmos DB para la versión 1.x de Functions se proporcionan en el paquete NuGet [Microsoft.Azure.WebJobs.Extensions.DocumentDB](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.DocumentDB), versión 1.x. El código fuente de los enlaces está en el repositorio [azure-webjobs-sdk-extensions](https://github.com/Azure/azure-webjobs-sdk-extensions/tree/v2.x/src/WebJobs.Extensions.DocumentDB) de GitHub.

[!INCLUDE [functions-package](../../includes/functions-package.md)]

## <a name="trigger"></a>Desencadenador

El desencadenador de Azure Cosmos DB utiliza la [fuente de cambios de Azure Cosmos DB](../cosmos-db/change-feed.md) para estar atento a las inserciones y actualizaciones de las particiones. La fuente de cambios publica inserciones y actualizaciones, no eliminaciones.

## <a name="trigger---example"></a>Desencadenador: ejemplo

# <a name="c"></a>[C#](#tab/csharp)

En el ejemplo siguiente se muestra un [función de C#](functions-dotnet-class-library.md) que se invoca cuando hay inserciones y actualizaciones en la base de datos y colección especificadas.

```cs
using Microsoft.Azure.Documents;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using System.Collections.Generic;

namespace CosmosDBSamplesV1
{
    public static class CosmosTrigger
    {
        [FunctionName("CosmosTrigger")]
        public static void Run([CosmosDBTrigger(
            databaseName: "ToDoItems",
            collectionName: "Items",
            ConnectionStringSetting = "CosmosDBConnection",
            LeaseCollectionName = "leases",
            CreateLeaseCollectionIfNotExists = true)]IReadOnlyList<Document> documents,
            TraceWriter log)
        {
            if (documents != null && documents.Count > 0)
            {
                log.Info($"Documents modified: {documents.Count}");
                log.Info($"First document Id: {documents[0].Id}");
            }
        }
    }
}
```

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

En el ejemplo siguiente se muestra un enlace de desencadenador de Cosmos DB en un archivo *function.json* y una [función de script de C#](functions-reference-csharp.md) que usa el enlace. La función escribe mensajes de registro cuando se modifican los registros de Cosmos DB.

Estos son los datos de enlace del archivo *function.json*:

```json
{
    "type": "cosmosDBTrigger",
    "name": "documents",
    "direction": "in",
    "leaseCollectionName": "leases",
    "connectionStringSetting": "<connection-app-setting>",
    "databaseName": "Tasks",
    "collectionName": "Items",
    "createLeaseCollectionIfNotExists": true
}
```

Este es el código de script de C#:

```cs
    #r "Microsoft.Azure.Documents.Client"
    
    using System;
    using Microsoft.Azure.Documents;
    using System.Collections.Generic;
    

    public static void Run(IReadOnlyList<Document> documents, TraceWriter log)
    {
        log.Info("Documents modified " + documents.Count);
        log.Info("First document Id " + documents[0].Id);
    }
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

En el ejemplo siguiente se muestra un enlace de desencadenador de Cosmos DB en un archivo *function.json* y una [función de JavaScript](functions-reference-node.md) que usa el enlace. La función escribe mensajes de registro cuando se modifican los registros de Cosmos DB.

Estos son los datos de enlace del archivo *function.json*:

```json
{
    "type": "cosmosDBTrigger",
    "name": "documents",
    "direction": "in",
    "leaseCollectionName": "leases",
    "connectionStringSetting": "<connection-app-setting>",
    "databaseName": "Tasks",
    "collectionName": "Items",
    "createLeaseCollectionIfNotExists": true
}
```

Este es el código de JavaScript:

```javascript
    module.exports = function (context, documents) {
        context.log('First document Id modified : ', documents[0].id);

        context.done();
    }
```

---

## <a name="trigger---attributes"></a>Desencadenador: atributos

# <a name="c"></a>[C#](#tab/csharp)

En las [bibliotecas de clases de C#](functions-dotnet-class-library.md), use el atributo [CosmosDBTrigger](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.CosmosDB/Trigger/CosmosDBTriggerAttribute.cs).

El constructor del atributo toma el nombre de la base de datos y el nombre de la colección. Para información sobre esos valores y otras propiedades que puede configurar, consulte [Desencadenador: configuración](#trigger---configuration). A continuación, se muestra un ejemplo del atributo `CosmosDBTrigger` en una signatura de método:

```csharp
    [FunctionName("DocumentUpdates")]
    public static void Run(
        [CosmosDBTrigger("database", "collection", ConnectionStringSetting = "myCosmosDB")]
    IReadOnlyList<Document> documents,
        TraceWriter log)
    {
        ...
    }
```

Para un ejemplo completo, consulte [Desencadenador: ejemplo de C#](#trigger).

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

El script de C# no admite atributos.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

JavaScript no admite atributos.

---

## <a name="trigger---configuration"></a>Desencadenador: configuración

En la siguiente tabla se explican las propiedades de configuración de enlace que se definen en el archivo *function.json* y el atributo `CosmosDBTrigger`.

|Propiedad de function.json | Propiedad de atributo |Descripción|
|---------|---------|----------------------|
|**type** | N/D | Se debe establecer en `cosmosDBTrigger`. |
|**direction** | N/D | Se debe establecer en `in`. Este parámetro se establece automáticamente cuando se crea el desencadenador en Azure Portal. |
|**name** | N/D | Nombre de la variable que se utiliza en el código de función y que representa la lista de documentos con los cambios. |
|**connectionStringSetting**|**ConnectionStringSetting** | Nombre de una configuración de aplicación que contiene la cadena de conexión utilizada para conectarse a la cuenta de Azure Cosmos DB que se está supervisando. |
|**databaseName**|**DatabaseName**  | Nombre de la base de datos de Azure Cosmos DB con la colección que se está supervisando. |
|**collectionName** |**CollectionName** | Nombre de la colección que se está supervisando. |
|**leaseConnectionStringSetting** | **LeaseConnectionStringSetting** | (Opcional) Nombre de una configuración de aplicación que contiene la cadena de conexión al servicio que incluye la colección de concesiones. Si no se establece, se usa el valor `connectionStringSetting`. Este parámetro se establece automáticamente cuando se crea el enlace en el portal. La cadena de conexión para la colección de concesiones debe tener permisos de escritura.|
|**leaseDatabaseName** |**LeaseDatabaseName** | (Opcional) Nombre de la base de datos que contiene la colección que se usa para almacenar las concesiones. Si no se establece, se usa el valor de la configuración `databaseName`. Este parámetro se establece automáticamente cuando se crea el enlace en el portal. |
|**leaseCollectionName** | **LeaseCollectionName** | (Opcional) Nombre de la colección que se usa para almacenar concesiones. Si no se establece, se usa el valor `leases`. |
|**createLeaseCollectionIfNotExists** | **CreateLeaseCollectionIfNotExists** | (Opcional) Cuando se establece en `true`, la colección de concesiones se crea automáticamente cuando todavía no existe. El valor predeterminado es `false`. |
|**leasesCollectionThroughput**| **LeasesCollectionThroughput**| (Opcional) Define la cantidad de unidades de solicitud que se asignan cuando se crea la colección de concesiones. Esta configuración solo se usa cuando `createLeaseCollectionIfNotExists` se establece en `true`. Este parámetro se establece automáticamente cuando el enlace se crea con el portal.
|**leaseCollectionPrefix**| **LeaseCollectionPrefix**| (Opcional) Al establecerse, agrega un prefijo a las concesiones creadas en la recopilación de concesiones para esta función, permitiendo de forma eficaz a dos funciones de Azure independientes compartir la misma recopilación de concesiones con prefijos diferentes.
|**feedPollDelay**| **FeedPollDelay**| (Opcional) Al establecerse, define, en milisegundos, el retraso entre sondeos de una partición en busca de nuevos cambios en la fuente, después de que todos los cambios actuales se purguen. El valor predeterminado es 5000 (5 segundos).
|**leaseAcquireInterval**| **LeaseAcquireInterval**| (Opcional) Al establecerse, define, en milisegundos, el intervalo para iniciar una tarea para calcular si las particiones se distribuyen uniformemente entre las instancias de host conocidas. El valor predeterminado es 13 000 (13 segundos).
|**leaseExpirationInterval**| **LeaseExpirationInterval**| (Opcional) Al establecerse, define, en milisegundos, el intervalo para el que se toma la concesión en una concesión que representa una partición. Si la concesión no se renueva dentro de este intervalo, expirará y la propiedad de la partición se moverá a otra instancia. El valor predeterminado es 60 000 (60 segundos).
|**leaseRenewInterval**| **LeaseRenewInterval**| (Opcional) Al establecerse, define, en milisegundos, el intervalo de renovación para todas las concesiones para las particiones que mantiene una instancia actualmente. El valor predeterminado es 17 000 (17 segundos).
|**checkpointFrequency**| **CheckpointFrequency**| (Opcional) Al establecerse, define, en milisegundos, el intervalo entre los puntos de comprobación de las concesiones. El valor predeterminado siempre se encuentra después de una llamada de función.
|**maxItemsPerInvocation**| **MaxItemsPerInvocation**| (Opcional) Al establecerse, personaliza la cantidad máxima de elementos recibidos por la llamada de función.
|**startFromBeginning**| **StartFromBeginning**| (Opcional) Cuando se establece, indica al desencadenador que comience a leer los cambios desde el principio del historial de la colección en lugar de desde la hora actual. Esto solo funciona la primera vez que se inicia el desencadenador, ya que en las ejecuciones posteriores, los puntos de control ya están almacenados. Si esta opción se establece en `true` cuando hay concesiones creadas no tiene ningún efecto.

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="trigger---usage"></a>Desencadenador: uso

El desencadenador requiere una segunda colección que utiliza para almacenar _concesiones_ en las particiones. La colección que se está supervisando y la colección que contiene las concesiones deben estar disponibles para que el desencadenador funcione.

>[!IMPORTANT]
> Si hay varias funciones configuradas para usar un desencadenador de Cosmos DB para la misma colección, cada una de ellas debe usar una colección de concesión dedicada, o bien se debe especificar un valor de `LeaseCollectionPrefix` para cada función. En caso contrario, se desencadenará solo una de las funciones. Para obtener información acerca del prefijo, consulte la [sección de configuración](#trigger---configuration).

El desencadenador no indica si un documento se actualizó o se insertó; solo proporciona el propio documento. Si tiene que administrar las actualizaciones e inserciones de manera diferente, puede hacerlo mediante la implementación de campos de marca de tiempo para las inserciones o actualizaciones.

## <a name="input"></a>Entrada

El enlace de entrada de Azure Cosmos DB usa SQL API para recuperar uno o varios documentos de Azure Cosmos DB y los pasa al parámetro de entrada de la función. Se puede determinar el identificador de documento o los parámetros de consulta según el desencadenador que invoca la función.

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
namespace CosmosDBSamplesV1
{
    public class ToDoItem
    {
        public string Id { get; set; }
        public string Description { get; set; }
    }
}
```

<a id="queue-trigger-look-up-id-from-json-c"></a>

### <a name="queue-trigger-look-up-id-from-json"></a>Desencadenador de cola, buscar identificador a partir de un JSON

En el ejemplo siguiente se muestra una [función de C#](functions-dotnet-class-library.md) que recupera un documento individual. La función la desencadena un mensaje de la cola que contiene un objeto JSON. El desencadenador de la cola analiza el JSON en un objeto denominado `ToDoItemLookup`, que contiene el identificador que se va a buscar. Dicho identificador se usa para recuperar un documento de `ToDoItem` de la base de datos y colección especificadas.

```cs
namespace CosmosDBSamplesV1
{
    public class ToDoItemLookup
    {
        public string ToDoItemId { get; set; }
    }
}
```

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;

namespace CosmosDBSamplesV1
{
    public static class DocByIdFromJSON
    {
        [FunctionName("DocByIdFromJSON")]
        public static void Run(
            [QueueTrigger("todoqueueforlookup")] ToDoItemLookup toDoItemLookup,
            [DocumentDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection",
                Id = "{ToDoItemId}")]ToDoItem toDoItem,
            TraceWriter log)
        {
            log.Info($"C# Queue trigger function processed Id={toDoItemLookup?.ToDoItemId}");

            if (toDoItem == null)
            {
                log.Info($"ToDo item not found");
            }
            else
            {
                log.Info($"Found ToDo item, Description={toDoItem.Description}");
            }
        }
    }
}
```

<a id="http-trigger-look-up-id-from-query-string-c"></a>

### <a name="http-trigger-look-up-id-from-query-string"></a>Desencadenador de HTTP, buscar identificador a partir de una cadena de consulta

En el ejemplo siguiente se muestra una [función de C#](functions-dotnet-class-library.md) que recupera un documento individual. La función la desencadena una solicitud HTTP que utiliza una cadena de consulta para especificar el identificador que se va a buscar. Dicho identificador se usa para recuperar un documento de `ToDoItem` de la base de datos y colección especificadas.

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs.Host;
using System.Net;
using System.Net.Http;

namespace CosmosDBSamplesV1
{
    public static class DocByIdFromQueryString
    {
        [FunctionName("DocByIdFromQueryString")]
        public static HttpResponseMessage Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)]HttpRequestMessage req,
            [DocumentDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection",
                Id = "{Query.id}")] ToDoItem toDoItem,
            TraceWriter log)
        {
            log.Info("C# HTTP trigger function processed a request.");
            if (toDoItem == null)
            {
                log.Info($"ToDo item not found");
            }
            else
            {
                log.Info($"Found ToDo item, Description={toDoItem.Description}");
            }
            return req.CreateResponse(HttpStatusCode.OK);
        }
    }
}
```

<a id="http-trigger-look-up-id-from-route-data-c"></a>

### <a name="http-trigger-look-up-id-from-route-data"></a>Desencadenador de HTTP, buscar identificador a partir de datos de ruta

En el ejemplo siguiente se muestra una [función de C#](functions-dotnet-class-library.md) que recupera un documento individual. La función la desencadena una solicitud HTTP que utiliza los datos de la ruta para especificar el identificador que se va a buscar. Dicho identificador se usa para recuperar un documento de `ToDoItem` de la base de datos y colección especificadas.

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs.Host;
using System.Net;
using System.Net.Http;

namespace CosmosDBSamplesV1
{
    public static class DocByIdFromRouteData
    {
        [FunctionName("DocByIdFromRouteData")]
        public static HttpResponseMessage Run(
            [HttpTrigger(
                AuthorizationLevel.Anonymous, "get", "post",
                Route = "todoitems/{id}")]HttpRequestMessage req,
            [DocumentDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection",
                Id = "{id}")] ToDoItem toDoItem,
            TraceWriter log)
        {
            log.Info("C# HTTP trigger function processed a request.");

            if (toDoItem == null)
            {
                log.Info($"ToDo item not found");
            }
            else
            {
                log.Info($"Found ToDo item, Description={toDoItem.Description}");
            }
            return req.CreateResponse(HttpStatusCode.OK);
        }
    }
}
```

[Omisión de los ejemplos de entrada](#input---attributes)

<a id="http-trigger-look-up-id-from-route-data-using-sqlquery-c"></a>

### <a name="http-trigger-look-up-id-from-route-data-using-sqlquery"></a>Desencadenador de HTTP, buscar identificador a partir de datos de ruta mediante SqlQuery

En el ejemplo siguiente se muestra una [función de C#](functions-dotnet-class-library.md) que recupera un documento individual. La función la desencadena una solicitud HTTP que utiliza los datos de la ruta para especificar el identificador que se va a buscar. Dicho identificador se usa para recuperar un documento de `ToDoItem` de la base de datos y colección especificadas.

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs.Host;
using System.Collections.Generic;
using System.Net;
using System.Net.Http;

namespace CosmosDBSamplesV1
{
    public static class DocByIdFromRouteDataUsingSqlQuery
    {
        [FunctionName("DocByIdFromRouteDataUsingSqlQuery")]
        public static HttpResponseMessage Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post",
                Route = "todoitems2/{id}")]HttpRequestMessage req,
            [DocumentDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection",
                SqlQuery = "select * from ToDoItems r where r.id = {id}")] IEnumerable<ToDoItem> toDoItems,
            TraceWriter log)
        {
            log.Info("C# HTTP trigger function processed a request.");
            foreach (ToDoItem toDoItem in toDoItems)
            {
                log.Info(toDoItem.Description);
            }
            return req.CreateResponse(HttpStatusCode.OK);
        }
    }
}
```

[Omisión de los ejemplos de entrada](#input---attributes)

<a id="http-trigger-get-multiple-docs-using-sqlquery-c"></a>

### <a name="http-trigger-get-multiple-docs-using-sqlquery"></a>Desencadenador de HTTP, obtener varios documentos mediante SqlQuery

En el ejemplo siguiente se muestra una [función de C#](functions-dotnet-class-library.md) que recupera una lista de documentos. La función la desencadena una solicitud HTTP. La consulta se especifica en la propiedad del atributo `SqlQuery`.

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs.Host;
using System.Collections.Generic;
using System.Net;
using System.Net.Http;

namespace CosmosDBSamplesV1
{
    public static class DocsBySqlQuery
    {
        [FunctionName("DocsBySqlQuery")]
        public static HttpResponseMessage Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)]
                HttpRequestMessage req,
            [DocumentDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection",
                SqlQuery = "SELECT top 2 * FROM c order by c._ts desc")]
                IEnumerable<ToDoItem> toDoItems,
            TraceWriter log)
        {
            log.Info("C# HTTP trigger function processed a request.");
            foreach (ToDoItem toDoItem in toDoItems)
            {
                log.Info(toDoItem.Description);
            }
            return req.CreateResponse(HttpStatusCode.OK);
        }
    }
}
```

[Omisión de los ejemplos de entrada](#input---attributes)

<a id="http-trigger-get-multiple-docs-using-documentclient-c"></a>

### <a name="http-trigger-get-multiple-docs-using-documentclient-c"></a>Desencadenador de HTTP, obtener varios documentos, mediante DocumentClient (C#)

En el ejemplo siguiente se muestra una [función de C#](functions-dotnet-class-library.md) que recupera una lista de documentos. La función la desencadena una solicitud HTTP. El código usa una instancia de `DocumentClient` que proporciona el enlace de Azure Cosmos DB para leer una lista de documentos. La instancia de `DocumentClient` también se puede usar para las operaciones de escritura.

```cs
using Microsoft.Azure.Documents.Client;
using Microsoft.Azure.Documents.Linq;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs.Host;
using System;
using System.Linq;
using System.Net;
using System.Net.Http;
using System.Threading.Tasks;

namespace CosmosDBSamplesV1
{
    public static class DocsByUsingDocumentClient
    {
        [FunctionName("DocsByUsingDocumentClient")]
        public static async Task<HttpResponseMessage> Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)]HttpRequestMessage req,
            [DocumentDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection")] DocumentClient client,
            TraceWriter log)
        {
            log.Info("C# HTTP trigger function processed a request.");

            Uri collectionUri = UriFactory.CreateDocumentCollectionUri("ToDoItems", "Items");
            string searchterm = req.GetQueryNameValuePairs()
                .FirstOrDefault(q => string.Compare(q.Key, "searchterm", true) == 0)
                .Value;

            if (searchterm == null)
            {
                return req.CreateResponse(HttpStatusCode.NotFound);
            }

            log.Info($"Searching for word: {searchterm} using Uri: {collectionUri.ToString()}");
            IDocumentQuery<ToDoItem> query = client.CreateDocumentQuery<ToDoItem>(collectionUri)
                .Where(p => p.Description.Contains(searchterm))
                .AsDocumentQuery();

            while (query.HasMoreResults)
            {
                foreach (ToDoItem result in await query.ExecuteNextAsync())
                {
                    log.Info(result.Description);
                }
            }
            return req.CreateResponse(HttpStatusCode.OK);
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
namespace CosmosDBSamplesV1
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
    "type": "documentDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "id" : "{queueTrigger}",
    "partitionKey": "{partition key value}",
    "connection": "MyAccount_COSMOSDB",
    "direction": "in"
}
```

En la sección de [configuración](#input---configuration) se explican estas propiedades.

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
    "type": "documentdb",
    "direction": "in",
    "databaseName": "MyDb",
    "collectionName": "MyCollection",
    "sqlQuery": "SELECT * from c where c.departmentId = {departmentId}",
    "connection": "CosmosDBConnection"
}
```

En la sección de [configuración](#input---configuration) se explican estas propiedades.

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

En el ejemplo siguiente se muestra una [función de script de C#](functions-reference-csharp.md) que recupera un documento individual. La función la desencadena una solicitud HTTP que utiliza una cadena de consulta para especificar el identificador que se va a buscar. Dicho identificador se usa para recuperar un documento de `ToDoItem` de la base de datos y colección especificadas.

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
      "type": "documentDB",
      "name": "toDoItem",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connection": "CosmosDBConnection",
      "direction": "in",
      "Id": "{Query.id}"
    }
  ],
  "disabled": true
}
```

Este es el código de script de C#:

```cs
using System.Net;

public static HttpResponseMessage Run(HttpRequestMessage req, ToDoItem toDoItem, TraceWriter log)
{
    log.Info("C# HTTP trigger function processed a request.");

    if (toDoItem == null)
    {
        log.Info($"ToDo item not found");
    }
    else
    {
        log.Info($"Found ToDo item, Description={toDoItem.Description}");
    }
    return req.CreateResponse(HttpStatusCode.OK);
}
```

<a id="http-trigger-look-up-id-from-route-data-c-script"></a>

### <a name="http-trigger-look-up-id-from-route-data"></a>Desencadenador de HTTP, buscar identificador a partir de datos de ruta

En el ejemplo siguiente se muestra una [función de script de C#](functions-reference-csharp.md) que recupera un documento individual. La función la desencadena una solicitud HTTP que utiliza los datos de la ruta para especificar el identificador que se va a buscar. Dicho identificador se usa para recuperar un documento de `ToDoItem` de la base de datos y colección especificadas.

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
      "route":"todoitems/{id}"
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "type": "documentDB",
      "name": "toDoItem",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connection": "CosmosDBConnection",
      "direction": "in",
      "Id": "{id}"
    }
  ],
  "disabled": false
}
```

Este es el código de script de C#:

```cs
using System.Net;

public static HttpResponseMessage Run(HttpRequestMessage req, ToDoItem toDoItem, TraceWriter log)
{
    log.Info("C# HTTP trigger function processed a request.");

    if (toDoItem == null)
    {
        log.Info($"ToDo item not found");
    }
    else
    {
        log.Info($"Found ToDo item, Description={toDoItem.Description}");
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
      "type": "documentDB",
      "name": "toDoItems",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connection": "CosmosDBConnection",
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

public static HttpResponseMessage Run(HttpRequestMessage req, IEnumerable<ToDoItem> toDoItems, TraceWriter log)
{
    log.Info("C# HTTP trigger function processed a request.");

    foreach (ToDoItem toDoItem in toDoItems)
    {
        log.Info(toDoItem.Description);
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
      "type": "documentDB",
      "name": "client",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connection": "CosmosDBConnection",
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

public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, DocumentClient client, TraceWriter log)
{
    log.Info("C# HTTP trigger function processed a request.");

    Uri collectionUri = UriFactory.CreateDocumentCollectionUri("ToDoItems", "Items");
    string searchterm = req.GetQueryNameValuePairs()
        .FirstOrDefault(q => string.Compare(q.Key, "searchterm", true) == 0)
        .Value;

    if (searchterm == null)
    {
        return req.CreateResponse(HttpStatusCode.NotFound);
    }

    log.Info($"Searching for word: {searchterm} using Uri: {collectionUri.ToString()}");
    IDocumentQuery<ToDoItem> query = client.CreateDocumentQuery<ToDoItem>(collectionUri)
        .Where(p => p.Description.Contains(searchterm))
        .AsDocumentQuery();

    while (query.HasMoreResults)
    {
        foreach (ToDoItem result in await query.ExecuteNextAsync())
        {
            log.Info(result.Description);
        }
    }
    return req.CreateResponse(HttpStatusCode.OK);
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

En esta sección se incluyen los ejemplos siguientes:

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
    "type": "documentDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "id" : "{queueTrigger_payload_property}",
    "partitionKey": "{queueTrigger_payload_property}",
    "connection": "MyAccount_COSMOSDB",
    "direction": "in"
},
{
    "name": "inputDocumentOut",
    "type": "documentDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "createIfNotExists": false,
    "partitionKey": "{queueTrigger_payload_property}",
    "connection": "MyAccount_COSMOSDB",
    "direction": "out"
}
```

En la sección de [configuración](#input---configuration) se explican estas propiedades.

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

En el ejemplo siguiente se muestra una [función de JavaScript](functions-reference-node.md) que recupera un documento individual. La función la desencadena una solicitud HTTP que utiliza una cadena de consulta para especificar el identificador que se va a buscar. Dicho identificador se usa para recuperar un documento de `ToDoItem` de la base de datos y colección especificadas.

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
      "type": "documentDB",
      "name": "toDoItem",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connection": "CosmosDBConnection",
      "direction": "in",
      "Id": "{Query.id}"
    }
  ],
  "disabled": true
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

En el ejemplo siguiente se muestra una [función de JavaScript](functions-reference-node.md) que recupera un documento individual. La función la desencadena una solicitud HTTP que utiliza una cadena de consulta para especificar el identificador que se va a buscar. Dicho identificador se usa para recuperar un documento de `ToDoItem` de la base de datos y colección especificadas.

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
      "route":"todoitems/{id}"
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "type": "documentDB",
      "name": "toDoItem",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connection": "CosmosDBConnection",
      "direction": "in",
      "Id": "{id}"
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
    "type": "documentdb",
    "direction": "in",
    "databaseName": "MyDb",
    "collectionName": "MyCollection",
    "sqlQuery": "SELECT * from c where c.departmentId = {departmentId}",
    "connection": "CosmosDBConnection"
}
```

En la sección de [configuración](#input---configuration) se explican estas propiedades.

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

---

## <a name="input---attributes"></a>Entrada: atributos

# <a name="c"></a>[C#](#tab/csharp)

En las [bibliotecas de clases de C#](functions-dotnet-class-library.md), use el atributo [DocumentDB](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/v2.x/src/WebJobs.Extensions.DocumentDB/DocumentDBAttribute.cs).

El constructor del atributo toma el nombre de la base de datos y el nombre de la colección. Para información sobre esos valores y otras propiedades que puede configurar, consulte [la sección de configuración siguiente](#input---configuration).

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

El script de C# no admite atributos.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

JavaScript no admite atributos.

---

## <a name="input---configuration"></a>Entrada: configuración

En la siguiente tabla se explican las propiedades de configuración de enlace que se definen en el archivo *function.json* y el atributo `DocumentDB`.

|Propiedad de function.json | Propiedad de atributo |Descripción|
|---------|---------|----------------------|
|**type**     | N/D | Se debe establecer en `documentdb`.        |
|**direction**     | N/D | Se debe establecer en `in`.         |
|**name**     | N/D | Nombre del parámetro de enlace que representa al documento en la función.  |
|**databaseName** |**DatabaseName** |Base de datos que contiene el documento.        |
|**collectionName** |**CollectionName** | Nombre de la colección que contiene el documento. |
|**id**    | **Id** | Identificador del documento que se va a recuperar. Esta propiedad es compatible con [expresiones de enlace](./functions-bindings-expressions-patterns.md). No establezca las propiedades **id** y **sqlQuery** a la vez. Si no establece alguna de ellas, se recupera toda la colección. |
|**sqlQuery**  |**SqlQuery**  | Consulta SQL de Azure Cosmos DB que se usa para recuperar varios documentos. La propiedad es compatible con los enlaces en tiempo de ejecución, como en este ejemplo: `SELECT * FROM c where c.departmentId = {departmentId}`. No establezca las propiedades **id** y **sqlQuery** a la vez. Si no establece alguna de ellas, se recupera toda la colección.|
|**connection**     |**ConnectionStringSetting**|Nombre de la configuración de aplicación que contiene la cadena de conexión de Azure Cosmos DB.        |
|**partitionKey**|**PartitionKey**|Especifica el valor de la clave de partición para la búsqueda. Puede incluir parámetros de enlace.|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="input---usage"></a>Uso de entradas

# <a name="c"></a>[C#](#tab/csharp)

Cuando se sale de la función correctamente, los cambios realizados en el documento de entrada mediante parámetros de entrada con nombre se guardan automáticamente.

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

Cuando se sale de la función correctamente, los cambios realizados en el documento de entrada mediante parámetros de entrada con nombre se guardan automáticamente.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Las actualizaciones no se realizan automáticamente al cerrar la función. Por el contrario, use `context.bindings.<documentName>In` y `context.bindings.<documentName>Out` para realizar las actualizaciones. Consulte el [ejemplo de entrada](#input).

---

## <a name="output"></a>Output

El enlace de salida de Azure Cosmos DB permite escribir un nuevo documento en una base de datos de Azure Cosmos DB mediante SQL API.

# <a name="c"></a>[C#](#tab/csharp)

En esta sección se incluyen los ejemplos siguientes:

* Desencadenador de cola, escribir un documento
* Desencadenador de cola, escribir documentos mediante `IAsyncCollector`

Los ejemplos hacen referencia a un tipo de `ToDoItem` simple:

```cs
namespace CosmosDBSamplesV1
{
    public class ToDoItem
    {
        public string Id { get; set; }
        public string Description { get; set; }
    }
}
```

### <a name="queue-trigger-write-one-doc"></a>Desencadenador de cola, escribir un documento

En el ejemplo siguiente se muestra una [función de C#](functions-dotnet-class-library.md) que agrega un documento a una base de datos mediante los datos que se proporcionan en el mensaje desde Queue Storage.

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using System;

namespace CosmosDBSamplesV1
{
    public static class WriteOneDoc
    {
        [FunctionName("WriteOneDoc")]
        public static void Run(
            [QueueTrigger("todoqueueforwrite")] string queueMessage,
            [DocumentDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection")]out dynamic document,
            TraceWriter log)
        {
            document = new { Description = queueMessage, id = Guid.NewGuid() };

            log.Info($"C# Queue trigger function inserted one row");
            log.Info($"Description={queueMessage}");
        }
    }
}
```

### <a name="queue-trigger-write-docs-using-iasynccollector"></a>Desencadenador de cola, escribir documentos mediante IAsyncCollector

En el ejemplo siguiente se muestra una [función de C#](functions-dotnet-class-library.md) que agrega una colección de documentos a una base de datos mediante los datos que se proporcionan en un mensaje de cola JSON.

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using System.Threading.Tasks;

namespace CosmosDBSamplesV1
{
    public static class WriteDocsIAsyncCollector
    {
        [FunctionName("WriteDocsIAsyncCollector")]
        public static async Task Run(
            [QueueTrigger("todoqueueforwritemulti")] ToDoItem[] toDoItemsIn,
            [DocumentDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection")]
                IAsyncCollector<ToDoItem> toDoItemsOut,
            TraceWriter log)
        {
            log.Info($"C# Queue trigger function processed {toDoItemsIn?.Length} items");

            foreach (ToDoItem toDoItem in toDoItemsIn)
            {
                log.Info($"Description={toDoItem.Description}");
                await toDoItemsOut.AddAsync(toDoItem);
            }
        }
    }
}
```

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

En esta sección se incluyen los ejemplos siguientes:

* Desencadenador de cola, escribir un documento
* Desencadenador de cola, escribir documentos mediante `IAsyncCollector`

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
    "type": "documentDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "createIfNotExists": true,
    "connection": "MyAccount_COSMOSDB",
    "direction": "out"
}
```

En la sección de [configuración](#output---configuration) se explican estas propiedades.

Este es el código de script de C#:

```cs
    #r "Newtonsoft.Json"

    using Microsoft.Azure.WebJobs.Host;
    using Newtonsoft.Json.Linq;

    public static void Run(string myQueueItem, out object employeeDocument, TraceWriter log)
    {
        log.Info($"C# Queue trigger function processed: {myQueueItem}");

        dynamic employee = JObject.Parse(myQueueItem);

        employeeDocument = new {
            id = employee.name + "-" + employee.employeeId,
            name = employee.name,
            employeeId = employee.employeeId,
            address = employee.address
        };
    }
```

### <a name="queue-trigger-write-docs-using-iasynccollector"></a>Desencadenador de cola, escribir documentos mediante IAsyncCollector

Para crear varios documentos, puede enlazar a `ICollector<T>` o `IAsyncCollector<T>`, donde `T` es uno de los tipos admitidos.

En este ejemplo se hace referencia a un tipo simple `ToDoItem`:

```cs
namespace CosmosDBSamplesV1
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
      "connection": "AzureWebJobsStorage"
    },
    {
      "type": "documentDB",
      "name": "toDoItemsOut",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connection": "CosmosDBConnection",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

Este es el código de script de C#:

```cs
using System;

public static async Task Run(ToDoItem[] toDoItemsIn, IAsyncCollector<ToDoItem> toDoItemsOut, TraceWriter log)
{
    log.Info($"C# Queue trigger function processed {toDoItemsIn?.Length} items");

    foreach (ToDoItem toDoItem in toDoItemsIn)
    {
        log.Info($"Description={toDoItem.Description}");
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
    "type": "documentDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "createIfNotExists": true,
    "connection": "MyAccount_COSMOSDB",
    "direction": "out"
}
```

En la sección de [configuración](#output---configuration) se explican estas propiedades.

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

---

## <a name="output---attributes"></a>Salida: atributos

# <a name="c"></a>[C#](#tab/csharp)

En las [bibliotecas de clases de C#](functions-dotnet-class-library.md), use el atributo [DocumentDB](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/v2.x/src/WebJobs.Extensions.DocumentDB/DocumentDBAttribute.cs).

El constructor del atributo toma el nombre de la base de datos y el nombre de la colección. Para información sobre esos valores y otras propiedades que puede configurar, consulte [Salida: configuración](#output---configuration). A continuación, se muestra un ejemplo del atributo `DocumentDB` en una signatura de método:

```csharp
    [FunctionName("QueueToDocDB")]
    public static void Run(
        [QueueTrigger("myqueue-items", Connection = "AzureWebJobsStorage")] string myQueueItem,
        [DocumentDB("ToDoList", "Items", Id = "id", ConnectionStringSetting = "myCosmosDB")] out dynamic document)
    {
        ...
    }
```

Para obtener un ejemplo completo, consulte la [salida](#output).

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

El script de C# no admite atributos.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

JavaScript no admite atributos.

---

## <a name="output---configuration"></a>Salida: configuración

En la siguiente tabla se explican las propiedades de configuración de enlace que se definen en el archivo *function.json* y el atributo `DocumentDB`.

|Propiedad de function.json | Propiedad de atributo |Descripción|
|---------|---------|----------------------|
|**type**     | N/D | Se debe establecer en `documentdb`.        |
|**direction**     | N/D | Se debe establecer en `out`.         |
|**name**     | N/D | Nombre del parámetro de enlace que representa al documento en la función.  |
|**databaseName** | **DatabaseName**|Base de datos que contiene la colección en la que se ha creado el documento.     |
|**collectionName** |**CollectionName**  | Nombre de la colección en la que se ha creado el documento. |
|**createIfNotExists**  |**CreateIfNotExists**    | Valor booleano que indica si la colección se ha creado si no existía. El valor predeterminado es *false* porque las colecciones nuevas se crean con rendimiento reservado, lo que afecta el costo. Consulte la [página de precios](https://azure.microsoft.com/pricing/details/documentdb/)para obtener más información.  |
|**partitionKey**|**PartitionKey** |Cuando el valor de `CreateIfNotExists` es true, define la ruta de la clave de partición para la colección que se creó.|
|**collectionThroughput**|**CollectionThroughput**| Cuando el valor de `CreateIfNotExists` es true, define el [rendimiento](../cosmos-db/set-throughput.md) de la colección que se creó.|
|**connection**    |**ConnectionStringSetting** |Nombre de la configuración de aplicación que contiene la cadena de conexión de Azure Cosmos DB.        |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="output---usage"></a>Uso de salidas

De forma predeterminada, cuando se escribe en el parámetro de salida de la función, se crea un documento en la base de datos. Este documento tiene un GUID generado automáticamente como identificador de documento. Puede especificar el identificador de documento del documento de salida si especifica la propiedad `id` en el objeto JSON pasado al parámetro de salida.

> [!Note]
> Cuando especifica el identificador de un documento existente, se sobrescribe con el nuevo documento de salida.

## <a name="exceptions-and-return-codes"></a>Excepciones y códigos de retorno

| Enlace | Referencia |
|---|---|
| CosmosDB | [Códigos de error de CosmosDB](/rest/api/cosmos-db/http-status-codes-for-cosmosdb) |

## <a name="next-steps"></a>Pasos siguientes

* [Más información sobre la informática de base de datos sin servidor con Cosmos DB](../cosmos-db/serverless-computing-database.md)
* [Más información sobre desencadenadores y enlaces de Azure Functions](functions-triggers-bindings.md)

<!---
> [!div class="nextstepaction"]
> [Go to a quickstart that uses a Cosmos DB trigger](functions-create-cosmos-db-triggered-function.md)
--->
