---
title: Flujos de cambios en la API de Azure Cosmos DB para MongoDB
description: Obtenga información sobre cómo usar los flujos de cambios en la API de Azure Cosmos DB para MongoDB con el fin de obtener los cambios realizados en sus datos.
author: Rodrigossz
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: how-to
ms.date: 06/04/2020
ms.author: rosouz
ms.custom: devx-track-javascript, devx-track-csharp
ms.openlocfilehash: 47f1fb4414b26ca9edbd992826a383ca65772ae7
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89003820"
---
# <a name="change-streams-in-azure-cosmos-dbs-api-for-mongodb"></a>Flujos de cambios en la API de Azure Cosmos DB para MongoDB

La compatibilidad con la [fuente de cambios](change-feed.md) en la API de Azure Cosmos DB para MongoDB está disponible mediante la API de flujos de cambios. Mediante el uso de la API de flujos de cambios, las aplicaciones pueden obtener los cambios realizados en la colección o en los elementos de una sola partición. Más adelante puede realizar otras acciones en función de los resultados. Los cambios en los elementos de la colección se capturan en el orden de su hora de modificación, y el criterio de ordenación se garantiza por clave de partición.

> [!NOTE]
> Para usar secuencias de cambios, cree la cuenta con la versión 3.6 de la API de Azure Cosmos DB para MongoDB, o una versión posterior. Si ejecuta ejemplos de secuencias de cambios con una versión anterior, es posible que vea el error `Unrecognized pipeline stage name: $changeStream`.

## <a name="examples"></a>Ejemplos

En el ejemplo siguiente se muestra cómo obtener flujos de cambios en todos los elementos de la colección. En este ejemplo se crea un cursor para inspeccionar los elementos cuando se insertan, actualizan o reemplazan. La fase `$match`, la fase `$project` y la opción `fullDocument` son necesarias para obtener los flujos de cambios. Actualmente no se admite la inspección de las operaciones de eliminación mediante flujos de cambios. Como solución alternativa, puede agregar un marcador flexible en los elementos que se van a eliminar. Por ejemplo, puede agregar un atributo en el elemento denominado "deleted". Si quiere eliminar el elemento, puede establecer "deleted" en `true` y establecer un TTL en el elemento. Ya que el cambio del elemento "deleted" a `true` es una actualización, este cambio será visible en el flujo de cambios.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
var cursor = db.coll.watch(
    [
        { $match: { "operationType": { $in: ["insert", "update", "replace"] } } },
        { $project: { "_id": 1, "fullDocument": 1, "ns": 1, "documentKey": 1 } }
    ],
    { fullDocument: "updateLookup" });

while (!cursor.isExhausted()) {
    if (cursor.hasNext()) {
        printjson(cursor.next());
    }
}
```

# <a name="c"></a>[C#](#tab/csharp)

```csharp
var pipeline = new EmptyPipelineDefinition<ChangeStreamDocument<BsonDocument>>()
    .Match(change => change.OperationType == ChangeStreamOperationType.Insert || change.OperationType == ChangeStreamOperationType.Update || change.OperationType == ChangeStreamOperationType.Replace)
    .AppendStage<ChangeStreamDocument<BsonDocument>, ChangeStreamDocument<BsonDocument>, BsonDocument>(
    "{ $project: { '_id': 1, 'fullDocument': 1, 'ns': 1, 'documentKey': 1 }}");

var options = new ChangeStreamOptions{
        FullDocument = ChangeStreamFullDocumentOption.UpdateLookup
    };

var enumerator = coll.Watch(pipeline, options).ToEnumerable().GetEnumerator();

while (enumerator.MoveNext()){
        Console.WriteLine(enumerator.Current);
    }

enumerator.Dispose();
```

# <a name="java"></a>[Java](#tab/java)

En el siguiente ejemplo se muestra cómo usar la funcionalidad de flujo de cambios en Java; para ver el ejemplo completo, vea este [repositorio de GitHub](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-java-changestream/blob/master/mongostream/src/main/java/com/azure/cosmos/mongostream/App.java). En este ejemplo también se muestra cómo usar el método `resumeAfter` para buscar todos los cambios de la última lectura. 

```java
Bson match = Aggregates.match(Filters.in("operationType", asList("update", "replace", "insert")));

// Pick the field you are most interested in
Bson project = Aggregates.project(fields(include("_id", "ns", "documentKey", "fullDocument")));

// This variable is for second example
BsonDocument resumeToken = null;

// Now time to build the pipeline
List<Bson> pipeline = Arrays.asList(match, project);

//#1 Simple example to seek changes

// Create cursor with update_lookup
MongoChangeStreamCursor<ChangeStreamDocument<org.bson.Document>> cursor = collection.watch(pipeline)
        .fullDocument(FullDocument.UPDATE_LOOKUP).cursor();

Document document = new Document("name", "doc-in-step-1-" + Math.random());
collection.insertOne(document);

while (cursor.hasNext()) {
    // There you go, we got the change document.
    ChangeStreamDocument<Document> csDoc = cursor.next();

    // Let is pick the token which will help us resuming
    // You can save this token in any persistent storage and retrieve it later
    resumeToken = csDoc.getResumeToken();
    //Printing the token
    System.out.println(resumeToken);
    
    //Printing the document.
    System.out.println(csDoc.getFullDocument());
    //This break is intentional but in real project feel free to remove it.
    break;
}

cursor.close();

```
---

## <a name="changes-within-a-single-shard"></a>Cambios en una sola partición

En el ejemplo siguiente se muestra cómo obtener los cambios en los elementos de una sola partición. En este ejemplo se obtienen los cambios de los elementos que tienen una clave de partición igual a "a" y el valor de clave de partición igual a "1". Es posible que distintos clientes lean los cambios de diferentes particiones en paralelo.

```javascript
var cursor = db.coll.watch(
    [
        { 
            $match: { 
                $and: [
                    { "fullDocument.a": 1 }, 
                    { "operationType": { $in: ["insert", "update", "replace"] } }
                ]
            }
        },
        { $project: { "_id": 1, "fullDocument": 1, "ns": 1, "documentKey": 1} }
    ],
    { fullDocument: "updateLookup" });

```

## <a name="current-limitations"></a>Limitaciones actuales

Se aplican las siguientes limitaciones al usar flujos de cambios:

* Todavía no se admiten las propiedades `operationType` y `updateDescription` en el documento de salida.
* Actualmente se admiten los tipos de operaciones `insert`, `update`y `replace`. No obstante, todavía no se admiten la operación de eliminación ni otros eventos.

Debido a estas limitaciones, se requieren la fase $match, la fase $project y la opción fullDocument, tal y como se muestra en los ejemplos anteriores.

A diferencia de la fuente de cambios de la API SQL de Azure Cosmos DB, no hay una [biblioteca de procesadores de fuente de cambios](change-feed-processor.md) independiente para usar los flujos de cambios, ni es necesaria para un contenedor de concesiones. Actualmente no se admiten los [desencadenadores de Azure Functions](change-feed-functions.md) para procesar flujos de cambios.

## <a name="error-handling"></a>Control de errores

Se admiten los siguientes mensajes y códigos de error cuando se utilizan flujos de cambios:

* **Código de error HTTP 16500**: cuando se limita el flujo de cambios, se devuelve una página vacía.

* **NamespaceNotFound (OperationType Invalidate)** : Si ejecuta el flujo de cambios en la colección que no existe o si se quita la colección, se devuelve un error `NamespaceNotFound`. Dado que no se puede devolver la propiedad `operationType` en el documento de salida, en lugar del error `operationType Invalidate`, se devuelve el error `NamespaceNotFound`.

## <a name="next-steps"></a>Pasos siguientes

* [Usar el período de vida para expirar automáticamente los datos en la API de Azure Cosmos DB para MongoDB](mongodb-time-to-live.md)
* [Indexación en la API de Azure Cosmos DB para MongoDB](mongodb-indexing.md)
