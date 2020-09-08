---
title: Administración de conflictos entre regiones en Azure Cosmos DB
description: Obtenga información sobre cómo administrar los conflictos en Azure Cosmos DB mediante la creación de una directiva de tipo el último escritor gana o una directiva de resolución de conflictos personalizada.
author: anfeldma-ms
ms.service: cosmos-db
ms.topic: how-to
ms.date: 06/11/2020
ms.author: anfeldma
ms.custom: devx-track-javascript, devx-track-csharp
ms.openlocfilehash: ec9e6bc7daad05284893ce3148f9a80f500c5cbb
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89019953"
---
# <a name="manage-conflict-resolution-policies-in-azure-cosmos-db"></a>Administración de directivas de resolución de conflictos en Azure Cosmos DB

Con las operaciones de escritura en varias regiones, cuando varios clientes escriben en el mismo elemento podrían producirse conflictos. Cuando se produce un conflicto de datos, se puede resolver mediante el uso de distintas directivas de resolución de conflictos. En este artículo se describe cómo administrar directivas de resolución de conflictos.

## <a name="create-a-last-writer-wins-conflict-resolution-policy"></a>Creación de una directiva de resolución de conflictos del tipo "el último en escribir gana"

En estos ejemplos se muestra cómo configurar un contenedor con una directiva de resolución de conflictos del tipo "el último en escribir gana". La ruta de acceso predeterminada para modo "el último en escribir gana" es el campo de marca de tiempo o la propiedad `_ts`. Para la API SQL, también se puede establecer en una ruta de acceso definida por el usuario con un tipo numérico. En un conflicto, gana el valor más alto. Si no se establece la ruta de acceso o la que se establece no es válida, se usará de forma predeterminada `_ts`. Los conflictos resueltos con esta directiva no se muestran en la fuente de conflictos. Todas las API pueden usar esta directiva.

### <a name="net-sdk"></a><a id="create-custom-conflict-resolution-policy-lww-dotnet"></a>SDK para .NET

# <a name="net-sdk-v2"></a>[SDK de .NET V2](#tab/dotnetv2)

```csharp
DocumentCollection lwwCollection = await createClient.CreateDocumentCollectionIfNotExistsAsync(
  UriFactory.CreateDatabaseUri(this.databaseName), new DocumentCollection
  {
      Id = this.lwwCollectionName,
      ConflictResolutionPolicy = new ConflictResolutionPolicy
      {
          Mode = ConflictResolutionMode.LastWriterWins,
          ConflictResolutionPath = "/myCustomId",
      },
  });
```

# <a name="net-sdk-v3"></a>[SDK para .NET V3](#tab/dotnetv3)

```csharp
Container container = await createClient.GetDatabase(this.databaseName)
    .CreateContainerIfNotExistsAsync(new ContainerProperties(this.lwwCollectionName, "/partitionKey")
    {
        ConflictResolutionPolicy = new ConflictResolutionPolicy()
        {
            Mode = ConflictResolutionMode.LastWriterWins,
            ResolutionPath = "/myCustomId",
        }
    });
```
---

### <a name="java-v4-sdk"></a><a id="create-custom-conflict-resolution-policy-lww-javav4"></a> SDK de Java V4

# <a name="async"></a>[Asincrónico](#tab/api-async)

   API asincrónica del SDK para Java V4 (Maven com.azure::azure-cosmos)

   [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=ManageConflictResolutionLWWAsync)]

# <a name="sync"></a>[Sincronizar](#tab/api-sync)

   API sincrónica del SDK para Java V4 (Maven com.azure::azure-cosmos)

   [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/sync/SampleDocumentationSnippets.java?name=ManageConflictResolutionLWWSync)]

--- 

### <a name="java-v2-sdks"></a><a id="create-custom-conflict-resolution-policy-lww-javav2"></a>SDK de Java V2

# <a name="async-java-v2-sdk"></a>[SDK de Java V2 asincrónico](#tab/async)

[SDK de Java V2 asincrónico](sql-api-sdk-async-java.md) (Maven [com.microsoft.azure::azure-cosmosdb](https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb))

```java
DocumentCollection collection = new DocumentCollection();
collection.setId(id);
ConflictResolutionPolicy policy = ConflictResolutionPolicy.createLastWriterWinsPolicy("/myCustomId");
collection.setConflictResolutionPolicy(policy);
DocumentCollection createdCollection = client.createCollection(databaseUri, collection, null).toBlocking().value();
```

# <a name="sync-java-v2-sdk"></a>[SDK de Java V2 sincrónico](#tab/sync)

[SDK de Java V2 sincrónico](sql-api-sdk-java.md) (Maven [com.microsoft.azure::azure-documentdb](https://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb))

```java
DocumentCollection lwwCollection = new DocumentCollection();
lwwCollection.setId(this.lwwCollectionName);
ConflictResolutionPolicy lwwPolicy = ConflictResolutionPolicy.createLastWriterWinsPolicy("/myCustomId");
lwwCollection.setConflictResolutionPolicy(lwwPolicy);
DocumentCollection createdCollection = this.tryCreateDocumentCollection(createClient, database, lwwCollection);
```
---

### <a name="nodejsjavascripttypescript-sdk"></a><a id="create-custom-conflict-resolution-policy-lww-javascript"></a>SDK para Node.js/JavaScript/TypeScript

```javascript
const database = client.database(this.databaseName);
const { container: lwwContainer } = await database.containers.createIfNotExists(
  {
    id: this.lwwContainerName,
    conflictResolutionPolicy: {
      mode: "LastWriterWins",
      conflictResolutionPath: "/myCustomId"
    }
  }
);
```

### <a name="python-sdk"></a><a id="create-custom-conflict-resolution-policy-lww-python"></a>SDK para Python

```python
udp_collection = {
    'id': self.udp_collection_name,
    'conflictResolutionPolicy': {
        'mode': 'LastWriterWins',
        'conflictResolutionPath': '/myCustomId'
    }
}
udp_collection = self.try_create_document_collection(
    create_client, database, udp_collection)
```

## <a name="create-a-custom-conflict-resolution-policy-using-a-stored-procedure"></a>Creación de una directiva de resolución de conflictos personalizada con un procedimiento almacenado

En estos ejemplos se muestran cómo configurar un contenedor con una directiva de resolución de conflictos personalizada con un procedimiento almacenado para resolver el conflicto. Estos conflictos no se mostrarán en la fuente de conflictos, salvo que haya un error en el procedimiento almacenado. Una vez que se crea una directiva en el contenedor, es necesario crear el procedimiento almacenado. En el ejemplo del SDK de .NET siguiente se muestra un ejemplo. Esta directiva solo se admite en Core (SQL) API.

### <a name="sample-custom-conflict-resolution-stored-procedure"></a>Ejemplo de un procedimiento almacenado de resolución de conflictos personalizada

Los procedimientos almacenados de resolución de conflictos personalizada deben implementarse mediante la signatura de función que se muestra a continuación. No es necesario que el nombre de la función coincida con el nombre usado al registrar el procedimiento almacenado con el contenedor, pero simplifica la nomenclatura. A continuación, se muestra una descripción de los parámetros que se deben implementar para este procedimiento almacenado.

- **incomingItem**: el elemento insertado o actualizado en la confirmación que está generando los conflictos. Es nulo para las operaciones de eliminación.
- **existingItem**: el elemento confirmado actualmente. Este valor no es NULL en una actualización ye s NULL en una inserción o eliminaciones.
- **isTombstone**: valor booleano que indica si incomingItem está en conflicto con un elemento eliminado anteriormente. Cuando es verdadero, existingItem también es nulo.
- **conflictingItems**: Matriz de la versión confirmada de todos los elementos del contenedor que están en conflicto con incomingItem en el identificador o cualquier otra propiedad de índice único.

> [!IMPORTANT]
> Al igual que con cualquier procedimiento almacenado, un procedimiento de resolución de conflictos personalizado puede acceder a los datos con la misma clave de partición y puede realizar cualquier operación de inserción, actualización o eliminación para resolver los conflictos.

Este procedimiento almacenado de ejemplo resuelve los conflictos mediante la selección del valor más bajo de la ruta de acceso `/myCustomId`.

```javascript
function resolver(incomingItem, existingItem, isTombstone, conflictingItems) {
  var collection = getContext().getCollection();

  if (!incomingItem) {
      if (existingItem) {

          collection.deleteDocument(existingItem._self, {}, function (err, responseOptions) {
              if (err) throw err;
          });
      }
  } else if (isTombstone) {
      // delete always wins.
  } else {
      if (existingItem) {
          if (incomingItem.myCustomId > existingItem.myCustomId) {
              return; // existing item wins
          }
      }

      var i;
      for (i = 0; i < conflictingItems.length; i++) {
          if (incomingItem.myCustomId > conflictingItems[i].myCustomId) {
              return; // existing conflict item wins
          }
      }

      // incoming item wins - clear conflicts and replace existing with incoming.
      tryDelete(conflictingItems, incomingItem, existingItem);
  }

  function tryDelete(documents, incoming, existing) {
      if (documents.length > 0) {
          collection.deleteDocument(documents[0]._self, {}, function (err, responseOptions) {
              if (err) throw err;

              documents.shift();
              tryDelete(documents, incoming, existing);
          });
      } else if (existing) {
          collection.replaceDocument(existing._self, incoming,
              function (err, documentCreated) {
                  if (err) throw err;
              });
      } else {
          collection.createDocument(collection.getSelfLink(), incoming,
              function (err, documentCreated) {
                  if (err) throw err;
              });
      }
  }
}
```

### <a name="net-sdk"></a><a id="create-custom-conflict-resolution-policy-stored-proc-dotnet"></a>SDK para .NET

# <a name="net-sdk-v2"></a>[SDK de .NET V2](#tab/dotnetv2)

```csharp
DocumentCollection udpCollection = await createClient.CreateDocumentCollectionIfNotExistsAsync(
  UriFactory.CreateDatabaseUri(this.databaseName), new DocumentCollection
  {
      Id = this.udpCollectionName,
      ConflictResolutionPolicy = new ConflictResolutionPolicy
      {
          Mode = ConflictResolutionMode.Custom,
          ConflictResolutionProcedure = string.Format("dbs/{0}/colls/{1}/sprocs/{2}", this.databaseName, this.udpCollectionName, "resolver"),
      },
  });

//Create the stored procedure
await clients[0].CreateStoredProcedureAsync(
UriFactory.CreateStoredProcedureUri(this.databaseName, this.udpCollectionName, "resolver"), new StoredProcedure
{
    Id = "resolver",
    Body = File.ReadAllText(@"resolver.js")
});
```

# <a name="net-sdk-v3"></a>[SDK para .NET V3](#tab/dotnetv3)

```csharp
Container container = await createClient.GetDatabase(this.databaseName)
    .CreateContainerIfNotExistsAsync(new ContainerProperties(this.udpCollectionName, "/partitionKey")
    {
        ConflictResolutionPolicy = new ConflictResolutionPolicy()
        {
            Mode = ConflictResolutionMode.Custom,
            ResolutionProcedure = string.Format("dbs/{0}/colls/{1}/sprocs/{2}", this.databaseName, this.udpCollectionName, "resolver")
        }
    });

await container.Scripts.CreateStoredProcedureAsync(
    new StoredProcedureProperties("resolver", File.ReadAllText(@"resolver.js"))
);
```
---

### <a name="java-v4-sdk"></a><a id="create-custom-conflict-resolution-policy-stored-proc-javav4"></a> SDK de Java V4

# <a name="async"></a>[Asincrónico](#tab/api-async)

   API asincrónica del SDK para Java V4 (Maven com.azure::azure-cosmos)

   [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=ManageConflictResolutionSprocAsync)]

# <a name="sync"></a>[Sincronizar](#tab/api-sync)

   API sincrónica del SDK para Java V4 (Maven com.azure::azure-cosmos)

   [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/sync/SampleDocumentationSnippets.java?name=ManageConflictResolutionSprocSync)]

--- 

### <a name="java-v2-sdks"></a><a id="create-custom-conflict-resolution-policy-stored-proc-javav2"></a>SDK de Java V2

# <a name="async-java-v2-sdk"></a>[SDK de Java V2 asincrónico](#tab/async)

[SDK de Java V2 asincrónico](sql-api-sdk-async-java.md) (Maven [com.microsoft.azure::azure-cosmosdb](https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb))

```java
DocumentCollection collection = new DocumentCollection();
collection.setId(id);
ConflictResolutionPolicy policy = ConflictResolutionPolicy.createCustomPolicy("resolver");
collection.setConflictResolutionPolicy(policy);
DocumentCollection createdCollection = client.createCollection(databaseUri, collection, null).toBlocking().value();
```

# <a name="sync-java-v2-sdk"></a>[SDK de Java V2 sincrónico](#tab/sync)

[SDK de Java V2 sincrónico](sql-api-sdk-java.md) (Maven [com.microsoft.azure::azure-documentdb](https://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb))

```java
DocumentCollection udpCollection = new DocumentCollection();
udpCollection.setId(this.udpCollectionName);
ConflictResolutionPolicy udpPolicy = ConflictResolutionPolicy.createCustomPolicy(
        String.format("dbs/%s/colls/%s/sprocs/%s", this.databaseName, this.udpCollectionName, "resolver"));
udpCollection.setConflictResolutionPolicy(udpPolicy);
DocumentCollection createdCollection = this.tryCreateDocumentCollection(createClient, database, udpCollection);
```
---

Una vez creado el contenedor, debe crear el procedimiento almacenado `resolver`.

### <a name="nodejsjavascripttypescript-sdk"></a><a id="create-custom-conflict-resolution-policy-stored-proc-javascript"></a>SDK para Node.js/JavaScript/TypeScript

```javascript
const database = client.database(this.databaseName);
const { container: udpContainer } = await database.containers.createIfNotExists(
  {
    id: this.udpContainerName,
    conflictResolutionPolicy: {
      mode: "Custom",
      conflictResolutionProcedure: `dbs/${this.databaseName}/colls/${
        this.udpContainerName
      }/sprocs/resolver`
    }
  }
);
```

Una vez creado el contenedor, debe crear el procedimiento almacenado `resolver`.

### <a name="python-sdk"></a><a id="create-custom-conflict-resolution-policy-stored-proc-python"></a>SDK para Python

```python
udp_collection = {
    'id': self.udp_collection_name,
    'conflictResolutionPolicy': {
        'mode': 'Custom',
        'conflictResolutionProcedure': 'dbs/' + self.database_name + "/colls/" + self.udp_collection_name + '/sprocs/resolver'
    }
}
udp_collection = self.try_create_document_collection(
    create_client, database, udp_collection)
```

Una vez creado el contenedor, debe crear el procedimiento almacenado `resolver`.

## <a name="create-a-custom-conflict-resolution-policy"></a>Creación de una directiva de resolución de conflictos personalizada

En estos ejemplos se muestran cómo configurar un contenedor con una directiva de resolución de conflictos personalizada. Estos conflictos se mostrarán en la fuente de conflictos.

### <a name="net-sdk"></a><a id="create-custom-conflict-resolution-policy-dotnet"></a>SDK para .NET

# <a name="net-sdk-v2"></a>[SDK de .NET V2](#tab/dotnetv2)

```csharp
DocumentCollection manualCollection = await createClient.CreateDocumentCollectionIfNotExistsAsync(
  UriFactory.CreateDatabaseUri(this.databaseName), new DocumentCollection
  {
      Id = this.manualCollectionName,
      ConflictResolutionPolicy = new ConflictResolutionPolicy
      {
          Mode = ConflictResolutionMode.Custom,
      },
  });
```

# <a name="net-sdk-v3"></a>[SDK para .NET V3](#tab/dotnetv3)

```csharp
Container container = await createClient.GetDatabase(this.databaseName)
    .CreateContainerIfNotExistsAsync(new ContainerProperties(this.manualCollectionName, "/partitionKey")
    {
        ConflictResolutionPolicy = new ConflictResolutionPolicy()
        {
            Mode = ConflictResolutionMode.Custom
        }
    });
```
---

### <a name="java-v4-sdk"></a><a id="create-custom-conflict-resolution-policy-javav4"></a> SDK de Java V4

# <a name="async"></a>[Asincrónico](#tab/api-async)

   API asincrónica del SDK para Java V4 (Maven com.azure::azure-cosmos)

   [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=ManageConflictResolutionCustomAsync)]

# <a name="sync"></a>[Sincronizar](#tab/api-sync)

   API sincrónica del SDK para Java V4 (Maven com.azure::azure-cosmos)

   [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/sync/SampleDocumentationSnippets.java?name=ManageConflictResolutionCustomSync)]

--- 

### <a name="java-v2-sdks"></a><a id="create-custom-conflict-resolution-policy-javav2"></a>SDK de Java V2

# <a name="async-java-v2-sdk"></a>[SDK de Java V2 asincrónico](#tab/async)

[SDK de Java V2 asincrónico](sql-api-sdk-async-java.md) (Maven [com.microsoft.azure::azure-cosmosdb](https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb))

```java
DocumentCollection collection = new DocumentCollection();
collection.setId(id);
ConflictResolutionPolicy policy = ConflictResolutionPolicy.createCustomPolicy();
collection.setConflictResolutionPolicy(policy);
DocumentCollection createdCollection = client.createCollection(databaseUri, collection, null).toBlocking().value();
```

# <a name="sync-java-v2-sdk"></a>[SDK de Java V2 sincrónico](#tab/sync)

[SDK de Java V2 sincrónico](sql-api-sdk-java.md) (Maven [com.microsoft.azure::azure-documentdb](https://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb))

```java
DocumentCollection manualCollection = new DocumentCollection();
manualCollection.setId(this.manualCollectionName);
ConflictResolutionPolicy customPolicy = ConflictResolutionPolicy.createCustomPolicy(null);
manualCollection.setConflictResolutionPolicy(customPolicy);
DocumentCollection createdCollection = client.createCollection(database.getSelfLink(), collection, null).getResource();
```
---

### <a name="nodejsjavascripttypescript-sdk"></a><a id="create-custom-conflict-resolution-policy-javascript"></a>SDK para Node.js/JavaScript/TypeScript

```javascript
const database = client.database(this.databaseName);
const {
  container: manualContainer
} = await database.containers.createIfNotExists({
  id: this.manualContainerName,
  conflictResolutionPolicy: {
    mode: "Custom"
  }
});
```

### <a name="python-sdk"></a><a id="create-custom-conflict-resolution-policy-python"></a>SDK para Python

```python
database = client.ReadDatabase("dbs/" + self.database_name)
manual_collection = {
    'id': self.manual_collection_name,
    'conflictResolutionPolicy': {
        'mode': 'Custom'
    }
}
manual_collection = client.CreateContainer(database['_self'], collection)
```

## <a name="read-from-conflict-feed"></a>Lectura desde la fuente de conflictos

En estos ejemplos se muestra cómo leer desde la fuente de conflictos de un contenedor. Los conflictos solo se muestran en la fuente si no se han resuelto de forma automática o si se usa una directiva de conflictos personalizada.

### <a name="net-sdk"></a><a id="read-from-conflict-feed-dotnet"></a>SDK para .NET

# <a name="net-sdk-v2"></a>[SDK de .NET V2](#tab/dotnetv2)

```csharp
FeedResponse<Conflict> conflicts = await delClient.ReadConflictFeedAsync(this.collectionUri);
```

# <a name="net-sdk-v3"></a>[SDK para .NET V3](#tab/dotnetv3)

```csharp
FeedIterator<ConflictProperties> conflictFeed = container.Conflicts.GetConflictQueryIterator();
while (conflictFeed.HasMoreResults)
{
    FeedResponse<ConflictProperties> conflicts = await conflictFeed.ReadNextAsync();
    foreach (ConflictProperties conflict in conflicts)
    {
        // Read the conflicted content
        MyClass intendedChanges = container.Conflicts.ReadConflictContent<MyClass>(conflict);
        MyClass currentState = await container.Conflicts.ReadCurrentAsync<MyClass>(conflict, new PartitionKey(intendedChanges.MyPartitionKey));

        // Do manual merge among documents
        await container.ReplaceItemAsync<MyClass>(intendedChanges, intendedChanges.Id, new PartitionKey(intendedChanges.MyPartitionKey));

        // Delete the conflict
        await container.Conflicts.DeleteAsync(conflict, new PartitionKey(intendedChanges.MyPartitionKey));
    }
}
```
---

### <a name="java-v2-sdks"></a><a id="read-from-conflict-feed-javav2"></a>SDK de Java V2

# <a name="async-java-v2-sdk"></a>[SDK de Java V2 asincrónico](#tab/async)

[SDK de Java V2 asincrónico](sql-api-sdk-async-java.md) (Maven [com.microsoft.azure::azure-cosmosdb](https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb))

```java
FeedResponse<Conflict> response = client.readConflicts(this.manualCollectionUri, null)
                    .first().toBlocking().single();
for (Conflict conflict : response.getResults()) {
    /* Do something with conflict */
}
```
# <a name="sync-java-v2-sdk"></a>[SDK de Java V2 sincrónico](#tab/sync)

[SDK de Java V2 sincrónico](sql-api-sdk-java.md) (Maven [com.microsoft.azure::azure-documentdb](https://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb))

```java
Iterator<Conflict> conflictsIterator = client.readConflicts(this.collectionLink, null).getQueryIterator();
while (conflictsIterator.hasNext()) {
    Conflict conflict = conflictsIterator.next();
    /* Do something with conflict */
}
```
---

### <a name="nodejsjavascripttypescript-sdk"></a><a id="read-from-conflict-feed-javascript"></a>SDK para Node.js/JavaScript/TypeScript

```javascript
const container = client
  .database(this.databaseName)
  .container(this.lwwContainerName);

const { result: conflicts } = await container.conflicts.readAll().toArray();
```

### <a name="python"></a><a id="read-from-conflict-feed-python"></a>Python

```python
conflicts_iterator = iter(client.ReadConflicts(self.manual_collection_link))
conflict = next(conflicts_iterator, None)
while conflict:
    # Do something with conflict
    conflict = next(conflicts_iterator, None)
```

## <a name="next-steps"></a>Pasos siguientes

Obtenga información acerca de los siguientes conceptos de Azure Cosmos DB:

- [Distribución global en segundo plano](global-dist-under-the-hood.md)
- [Configuración de una arquitectura multimaestro en las aplicaciones](how-to-multi-master.md)
- [Configuración de los clientes para el hospedaje múltiple](how-to-manage-database-account.md#configure-multiple-write-regions)
- [Incorporación o eliminación de regiones de una cuenta de Azure Cosmos DB](how-to-manage-database-account.md#addremove-regions-from-your-database-account)
- [Configuración de una arquitectura multimaestro en las aplicaciones](how-to-multi-master.md).
- [Creación de particiones y distribución de datos](partition-data.md)
- [Indexación en Azure Cosmos DB](indexing-policies.md)
