---
title: Creación de varios desencadenadores independientes de Azure Functions para Cosmos DB
description: Obtenga información sobre cómo configurar varios desencadenadores independientes de Azure Functions para Cosmos DB para crear arquitecturas controladas por eventos.
author: ealsur
ms.service: cosmos-db
ms.topic: how-to
ms.date: 07/17/2019
ms.author: maquaran
ms.custom: devx-track-csharp
ms.openlocfilehash: dce10fb85ac181bb06aef0058768bef659462a5a
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89019987"
---
# <a name="create-multiple-azure-functions-triggers-for-cosmos-db"></a>Cómo crear varios desencadenadores de Azure Functions para Cosmos DB

En este artículo se describe cómo puede configurar varios desencadenadores de Azure Functions para Cosmos DB para trabajar en paralelo y reaccionar de forma independiente a los cambios.

:::image type="content" source="./media/change-feed-functions/multi-trigger.png" alt-text="Funciones basadas en eventos sin servidor que funcionan con el desencadenador de Azure Functions para Cosmos DB y comparten un contenedor de concesiones" border="false":::

## <a name="event-based-architecture-requirements"></a>Requisitos de la arquitectura basada en eventos

Al compilar arquitecturas sin servidor con [Azure Functions](../azure-functions/functions-overview.md), se [recomienda](../azure-functions/functions-best-practices.md#avoid-long-running-functions) crear pequeños conjuntos de funciones que trabajen conjuntamente, en lugar de grandes funciones de larga duración.

Al compilar flujos sin servidor basados en eventos mediante el [desencadenador de Azure Functions para Cosmos DB](./change-feed-functions.md), se producirá un escenario en el que querrá realizar varias acciones siempre que haya un nuevo evento en un determinado [contenedor de Azure Cosmos](./databases-containers-items.md#azure-cosmos-containers). Si las acciones que quiere desencadenar son independientes entre sí, la solución ideal consiste en **crear un desencadenador de Azure Functions para Cosmos DB por cada acción** que quiera realizar y que todas escuchen los cambios en el mismo contenedor de Azure Cosmos.

## <a name="optimizing-containers-for-multiple-triggers"></a>Optimización de contenedores para varios desencadenadores

Dados los *requisitos* del desencadenador de Azure Functions para Cosmos DB, necesitamos un segundo contenedor para almacenar el estado, también denominado *contenedor de concesiones*. ¿Esto significa que necesita un contenedor de concesiones independiente para cada función de Azure?

Dispone de dos opciones:

* Crear **un contenedor de concesiones para cada función**: Este enfoque puede suponer costos adicionales, a menos que use una [base de datos de rendimiento compartido](./set-throughput.md#set-throughput-on-a-database). Recuerde que el rendimiento mínimo a nivel de contenedor es de 400 [unidades de solicitud](./request-units.md) y, en el caso del contenedor de concesiones, solo se usa para aplicar el punto de control al progreso y mantener el estado.
* Tener **un contenedor de concesiones y compartirlo** para todas las funciones: Esta segunda opción hace un mejor uso de las unidades de solicitud aprovisionadas en el contenedor, ya que permite que varias instancias de Azure Functions se compartan y usen el mismo rendimiento aprovisionado.

El objetivo de este artículo es guiarle con la segunda opción.

## <a name="configuring-a-shared-leases-container"></a>Configuración de un contenedor de concesiones compartido

Para configurar el contenedor de concesiones compartido, la única configuración adicional que tiene que realizar en los desencadenadores consiste en agregar el [atributo](../azure-functions/functions-bindings-cosmosdb-v2-trigger.md#attributes-and-annotations) `LeaseCollectionPrefix` si usa C# o el [atributo](../azure-functions/functions-bindings-cosmosdb-v2-trigger.md) `leaseCollectionPrefix` si usa JavaScript. El valor del atributo debe ser un descriptor lógico de ese desencadenador en particular.

Por ejemplo, si tiene tres desencadenadores (uno que envía correos electrónicos, otro que realiza una agregación para crear una vista materializada y otro que envía los cambios a otro almacenamiento para su análisis posterior), puede asignar el elemento `LeaseCollectionPrefix` de "emails" al primero de ellos, "materialized"al segundo y "analytics" al tercero.

Lo importante es que los tres desencadenadores **puedan usar la misma configuración de contenedor de concesiones** (cuenta, base de datos y nombre del contenedor).

Un ejemplo de código muy sencillo con el atributo `LeaseCollectionPrefix` en C# tendría el aspecto siguiente:

```cs
using Microsoft.Azure.Documents;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using System.Collections.Generic;
using Microsoft.Extensions.Logging;

[FunctionName("SendEmails")]
public static void SendEmails([CosmosDBTrigger(
    databaseName: "ToDoItems",
    collectionName: "Items",
    ConnectionStringSetting = "CosmosDBConnection",
    LeaseCollectionName = "leases",
    LeaseCollectionPrefix = "emails")]IReadOnlyList<Document> documents,
    ILogger log)
{
    ...
}

[FunctionName("MaterializedViews")]
public static void MaterializedViews([CosmosDBTrigger(
    databaseName: "ToDoItems",
    collectionName: "Items",
    ConnectionStringSetting = "CosmosDBConnection",
    LeaseCollectionName = "leases",
    LeaseCollectionPrefix = "materialized")]IReadOnlyList<Document> documents,
    ILogger log)
{
    ...
}
```

Para JavaScript, puede aplicar la configuración en el archivo `function.json` con el atributo `leaseCollectionPrefix`:

```json
{
    "type": "cosmosDBTrigger",
    "name": "documents",
    "direction": "in",
    "leaseCollectionName": "leases",
    "connectionStringSetting": "CosmosDBConnection",
    "databaseName": "ToDoItems",
    "collectionName": "Items",
    "leaseCollectionPrefix": "emails"
},
{
    "type": "cosmosDBTrigger",
    "name": "documents",
    "direction": "in",
    "leaseCollectionName": "leases",
    "connectionStringSetting": "CosmosDBConnection",
    "databaseName": "ToDoItems",
    "collectionName": "Items",
    "leaseCollectionPrefix": "materialized"
}
```

> [!NOTE]
> Supervise siempre las unidades de solicitud aprovisionadas en el contenedor de concesiones compartido. Cada desencadenador que lo comparta aumentará el consumo medio de rendimiento, por lo que es posible que tenga que aumentar el rendimiento aprovisionado a medida que aumente el número de instancias de Azure Functions que lo estén usando.

## <a name="next-steps"></a>Pasos siguientes

* Consulte la configuración completa del [desencadenador de Azure Functions para Cosmos DB](../azure-functions/functions-bindings-cosmosdb-v2-trigger.md#configuration)
* Compruebe la [lista de ejemplos](../azure-functions/functions-bindings-cosmosdb-v2-trigger.md) extendida para todos los idiomas.
* Visite las recetas sin servidor con el [repositorio de GitHub](https://github.com/ealsur/serverless-recipes/tree/master/cosmosdbtriggerscenarios) de Azure Functions y Azure Cosmos DB para obtener más ejemplos.
