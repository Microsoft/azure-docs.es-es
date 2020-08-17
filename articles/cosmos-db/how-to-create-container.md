---
title: Creación de un contenedor en Azure Cosmos DB
description: Aprenda a crear un contenedor en Azure Cosmos DB mediante Azure Portal, .NET, Java, Python, Node.js y otros SDK.
author: markjbrown
ms.service: cosmos-db
ms.topic: how-to
ms.date: 07/29/2020
ms.author: mjbrown
ms.custom: devx-track-azurecli
ms.openlocfilehash: 6e62e307b6c5f5de939505eae8850f0117782a69
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87498368"
---
# <a name="create-an-azure-cosmos-container"></a>Creación de un contenedor de Azure Cosmos

En este artículo se explican las distintas formas de crear un contenedor de Azure Cosmos (colección, tabla o grafo) mediante Azure Portal, la CLI de Azure, PowerShell o SDK compatibles. En este artículo se muestra cómo crear un contenedor, especificar la clave de partición y aprovisionar el rendimiento.

> [!NOTE]
> Cuando cree contenedores, tenga cuidado de no utilizar el mismo nombre en dos de ellos con distintas mayúsculas y minúsculas. Algunos componentes de la plataforma de Azure no distinguen mayúsculas de minúsculas y esto puede producir confusión o problemas con los datos de telemetría y las acciones que se realicen en los contenedores con estos nombres.

## <a name="create-a-container-using-azure-portal"></a>Creación de un contenedor mediante Azure Portal

### <a name="sql-api"></a><a id="portal-sql"></a>SQL API

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. [Cree una cuenta de Azure Cosmos](create-sql-api-dotnet.md#create-account) o seleccione una ya existente.

1. Abra el panel **Data Explorer** y seleccione **Nuevo contenedor**. Después, proporcione los detalles siguientes:

   * Indique si va a crear una nueva base de datos o a usar una existente.
   * Escriba un identificador de contenedor.
   * Escriba la clave de partición.
   * Escriba el rendimiento que se va a aprovisionar (por ejemplo, 1000 RU).
   * Seleccione **Aceptar**.

    :::image type="content" source="./media/how-to-create-container/partitioned-collection-create-sql.png" alt-text="Captura de pantalla del panel de Data Explorer con la opción Contenedor nuevo resaltada":::

### <a name="azure-cosmos-db-api-for-mongodb"></a><a id="portal-mongodb"></a>Azure Cosmos DB API para MongoDB

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. [Cree una cuenta de Azure Cosmos](create-mongodb-dotnet.md#create-a-database-account) o seleccione una ya existente.

1. Abra el panel **Data Explorer** y seleccione **Nuevo contenedor**. Después, proporcione los detalles siguientes:

   * Indique si va a crear una nueva base de datos o a usar una existente.
   * Escriba un identificador de contenedor.
   * Escriba una clave de partición.
   * Escriba el rendimiento que se va a aprovisionar (por ejemplo, 1000 RU).
   * Seleccione **Aceptar**.

    :::image type="content" source="./media/how-to-create-container/partitioned-collection-create-mongodb.png" alt-text="Captura de pantalla del cuadro de diálogo Agregar contenedor de Azure Cosmos DB API para MongoDB":::

### <a name="cassandra-api"></a><a id="portal-cassandra"></a>Cassandra API

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. [Cree una cuenta de Azure Cosmos](create-cassandra-dotnet.md#create-a-database-account) o seleccione una ya existente.

1. Abra el panel **Data Explorer** y seleccione **Nueva tabla**. Después, proporcione los detalles siguientes:

   * Indique si va a crear un nuevo espacio de claves o a usar uno existente.
   * Escriba un nombre de tabla.
   * Especifique las propiedades y una clave principal.
   * Escriba el rendimiento que se va a aprovisionar (por ejemplo, 1000 RU).
   * Seleccione **Aceptar**.

    :::image type="content" source="./media/how-to-create-container/partitioned-collection-create-cassandra.png" alt-text="Captura de pantalla del cuadro de diálogo Agregar tabla de Cassandra API":::

> [!NOTE]
> Con Cassandra API, la clave principal se usa como clave de partición.

### <a name="gremlin-api"></a><a id="portal-gremlin"></a>Gremlin API

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. [Cree una cuenta de Azure Cosmos](create-graph-dotnet.md#create-a-database-account) o seleccione una ya existente.

1. Abra el panel de **Data Explorer** y seleccione **New Graph** (Nuevo grafo). Después, proporcione los detalles siguientes:

   * Indique si va a crear una nueva base de datos o a usar una existente.
   * Escriba un identificador de grafo.
   * Seleccione la capacidad de almacenamiento **Sin límite**.
   * Escriba una clave de partición para los vértices.
   * Escriba el rendimiento que se va a aprovisionar (por ejemplo, 1000 RU).
   * Seleccione **Aceptar**.

    :::image type="content" source="./media/how-to-create-container/partitioned-collection-create-gremlin.png" alt-text="Captura de pantalla del cuadro de diálogo Agregar grafo de la API Gremlin":::

### <a name="table-api"></a><a id="portal-table"></a>Table API

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. [Cree una cuenta de Azure Cosmos](create-table-dotnet.md#create-a-database-account) o seleccione una ya existente.

1. Abra el panel **Data Explorer** y seleccione **Nueva tabla**. Después, proporcione los detalles siguientes:

   * Escriba un identificador de tabla.
   * Escriba el rendimiento que se va a aprovisionar (por ejemplo, 1000 RU).
   * Seleccione **Aceptar**.

    :::image type="content" source="./media/how-to-create-container/partitioned-collection-create-table.png" alt-text="Captura de pantalla del cuadro de diálogo Agregar tabla de Table API":::

> [!Note]
> Con Table API, la clave de partición se especifica cada vez que se agrega una nueva fila.

## <a name="create-a-container-using-azure-cli"></a>Creación de un contenedor mediante la CLI de Azure<a id="cli-sql"></a><a id="cli-mongodb"></a><a id="cli-cassandra"></a><a id="cli-gremlin"></a><a id="cli-table"></a>

Los vínculos siguientes muestran cómo crear recursos de contenedor para Azure Cosmos DB mediante la CLI de Azure.

Para obtener una lista de todos los ejemplos de la CLI de Azure en todas las API de Azure Cosmos DB, vea [Ejemplos de la CLI de Azure para Azure Cosmos DB](cli-samples.md).

* [Creación de un contenedor con la CLI de Azure](manage-with-cli.md#create-a-container)
* [Creación de una colección para Azure Cosmos DB para MongoDB API con la CLI de Azure](./scripts/cli/mongodb/create.md)
* [Creación de una tabla de Cassandra con la CLI de Azure](./scripts/cli/cassandra/create.md)
* [Creación de un grafo de Gremlin con la CLI de Azure](./scripts/cli/gremlin/create.md)
* [Creación una tabla de Table API con la CLI de Azure](./scripts/cli/table/create.md)

## <a name="create-a-container-using-powershell"></a>Creación de un contenedor mediante PowerShell

En los vínculos siguientes se muestra cómo crear recursos de contenedor para Azure Cosmos DB mediante PowerShell.

Para obtener una lista de todos los ejemplos de PowerShell en todas las API de Azure Cosmos DB, vea [Ejemplos de PowerShell](powershell-samples.md).

* [Creación de un contenedor con PowerShell](manage-with-powershell.md#create-container)
* [Creación de una colección para Azure Cosmos DB para MongoDB API con PowerShell](./scripts/powershell/mongodb/create.md)
* [Creación de una tabla de Cassandra con PowerShell](./scripts/powershell/cassandra/create.md)
* [Creación de un grafo de Gremlin con PowerShell](./scripts/powershell/gremlin/create.md)
* [Creación de una tabla de Table API con PowerShell](./scripts/powershell/table/create.md)

## <a name="create-a-container-using-net-sdk"></a>Creación de un contenedor mediante el SDK para .NET

Si se produce una excepción de tiempo de espera al crear una colección, realice una operación de lectura para validar si la colección se ha creado correctamente. La operación de lectura emite una excepción hasta que la operación de creación de la colección se realiza correctamente. Para la lista de códigos de estado admitidos por la operación de creación, consulte el artículo [Códigos de estado HTTP para Azure Cosmos DB](/rest/api/cosmos-db/http-status-codes-for-cosmosdb).

### <a name="sql-api-and-gremlin-api"></a><a id="dotnet-sql-graph"></a>SQL API y Gremlin API

```csharp
// Create a container with a partition key and provision 1000 RU/s throughput.
DocumentCollection myCollection = new DocumentCollection();
myCollection.Id = "myContainerName";
myCollection.PartitionKey.Paths.Add("/myPartitionKey");

await client.CreateDocumentCollectionAsync(
    UriFactory.CreateDatabaseUri("myDatabaseName"),
    myCollection,
    new RequestOptions { OfferThroughput = 1000 });
```

### <a name="azure-cosmos-db-api-for-mongodb"></a><a id="dotnet-mongodb"></a>Azure Cosmos DB API para MongoDB

```csharp
// Create a collection with a partition key by using Mongo Shell:
db.runCommand( { shardCollection: "myDatabase.myCollection", key: { myShardKey: "hashed" } } )
```

> [!Note]
> El protocolo de conexión de MongoDB no entiende el concepto de [unidades de solicitud](request-units.md). Para crear una colección con rendimiento aprovisionado en ella, use Azure Portal o los SDK de Cosmos DB para la API de SQL.

### <a name="cassandra-api"></a><a id="dotnet-cassandra"></a>Cassandra API

```csharp
// Create a Cassandra table with a partition/primary key and provision 1000 RU/s throughput.
session.Execute(CREATE TABLE myKeySpace.myTable(
    user_id int PRIMARY KEY,
    firstName text,
    lastName text) WITH cosmosdb_provisioned_throughput=1000);
```

## <a name="next-steps"></a>Pasos siguientes

* [Creación de particiones en Azure Cosmos DB](partitioning-overview.md)
* [Unidades de solicitud en Azure Cosmos DB](request-units.md)
* [Aprovisionamiento del rendimiento en contenedores y bases de datos](set-throughput.md)
* [Uso de la cuenta de Azure Cosmos](account-overview.md)
