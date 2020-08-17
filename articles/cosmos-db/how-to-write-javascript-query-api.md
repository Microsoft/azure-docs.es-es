---
title: Escritura de procedimientos almacenados y desencadenadores con la API de consulta de JavaScript en Azure Cosmos DB
description: Aprenda a escribir procedimientos almacenados y desencadenadores con la API de consulta de Javascript en Azure Cosmos DB
author: timsander1
ms.service: cosmos-db
ms.topic: how-to
ms.date: 05/07/2020
ms.author: tisande
ms.custom: devx-track-javascript
ms.openlocfilehash: 2182e77ee673ea655762c765e2cfccb3decbe6af
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87420182"
---
# <a name="how-to-write-stored-procedures-and-triggers-in-azure-cosmos-db-by-using-the-javascript-query-api"></a>Escritura de procedimientos almacenados y desencadenadores en Azure Cosmos DB con la API de consulta de Javascript

Azure Cosmos DB permite realizar consultas optimizadas mediante una interfaz fluida de JavaScript sin ningún conocimiento del lenguaje de SQL que puede usarse para escribir procedimientos almacenados o desencadenadores. Para más información sobre la compatibilidad con la API de consulta de JavaScript en Azure Cosmos DB, consulte el artículo [Working with JavaScript language integrated query API in Azure Cosmos DB](javascript-query-api.md) (Trabajo con la API de consulta integrada de lenguaje de JavaScript en Azure Cosmos DB).

## <a name="stored-procedure-using-the-javascript-query-api"></a><a id="stored-procedures"></a>Procedimiento almacenado mediante la API de consulta de JavaScript

El ejemplo de código siguiente es un ejemplo de cómo se usa la API de consulta de JavaScript en el contexto de un procedimiento almacenado. El procedimiento almacenado inserta un elemento de Azure Cosmos especificado mediante un parámetro de entrada y actualiza un documento de metadatos mediante el método `__.filter()`, con los valores minSize, maxSize y totalSize basados en la propiedad de tamaño del elemento de entrada.

> [!NOTE]
> `__` (subrayado doble) es un alias para `getContext().getCollection()` cuando se usa JavaScript Query API.

```javascript
/**
 * Insert an item and update metadata doc: minSize, maxSize, totalSize based on item.size.
 */
function insertDocumentAndUpdateMetadata(item) {
  // HTTP error codes sent to our callback function by CosmosDB server.
  var ErrorCode = {
    RETRY_WITH: 449,
  }

  var isAccepted = __.createDocument(__.getSelfLink(), item, {}, function(err, item, options) {
    if (err) throw err;

    // Check the item (ignore items with invalid/zero size and metadata itself) and call updateMetadata.
    if (!item.isMetadata && item.size > 0) {
      // Get the metadata. We keep it in the same container. it's the only item that has .isMetadata = true.
      var result = __.filter(function(x) {
        return x.isMetadata === true
      }, function(err, feed, options) {
        if (err) throw err;

        // We assume that metadata item was pre-created and must exist when this script is called.
        if (!feed || !feed.length) throw new Error("Failed to find the metadata item.");

        // The metadata item.
        var metaItem = feed[0];

        // Update metaDoc.minSize:
        // for 1st document use doc.Size, for all the rest see if it's less than last min.
        if (metaItem.minSize == 0) metaItem.minSize = item.size;
        else metaItem.minSize = Math.min(metaItem.minSize, item.size);

        // Update metaItem.maxSize.
        metaItem.maxSize = Math.max(metaItem.maxSize, item.size);

        // Update metaItem.totalSize.
        metaItem.totalSize += item.size;

        // Update/replace the metadata item in the store.
        var isAccepted = __.replaceDocument(metaItem._self, metaItem, function(err) {
          if (err) throw err;
          // Note: in case concurrent updates causes conflict with ErrorCode.RETRY_WITH, we can't read the meta again
          //       and update again because due to Snapshot isolation we will read same exact version (we are in same transaction).
          //       We have to take care of that on the client side.
        });
        if (!isAccepted) throw new Error("replaceDocument(metaItem) returned false.");
      });
      if (!result.isAccepted) throw new Error("filter for metaItem returned false.");
    }
  });
  if (!isAccepted) throw new Error("createDocument(actual item) returned false.");
}
```

## <a name="next-steps"></a>Pasos siguientes

Consulte los artículos siguientes para obtener información sobre procedimientos almacenados, desencadenadores y funciones definidas por el usuario en Azure Cosmos DB:

* [How to work with stored procedures, triggers, user-defined functions in Azure Cosmos DB](how-to-use-stored-procedures-triggers-udfs.md) (Trabajo con procedimientos almacenados, desencadenadores y funciones definidas por el usuario en Azure Cosmos DB)

* [How to register and use stored procedures in Azure Cosmos DB](how-to-use-stored-procedures-triggers-udfs.md#stored-procedures) (Registro y uso de procedimientos almacenados en Azure Cosmos DB)

* How to register and use [pre-triggers](how-to-use-stored-procedures-triggers-udfs.md#pre-triggers) and [post-triggers](how-to-use-stored-procedures-triggers-udfs.md#post-triggers) in Azure Cosmos DB (Registro y uso de desencadenadores previos y posteriores en Azure Cosmos DB)

* [How to register and use user-defined functions in Azure Cosmos DB](how-to-use-stored-procedures-triggers-udfs.md#udfs) (Registro y uso de funciones definidas por el usuario en Azure Cosmos DB)

* [Synthetic partition keys in Azure Cosmos DB](synthetic-partition-keys.md) (Claves de partición sintéticas en Azure Cosmos DB)
