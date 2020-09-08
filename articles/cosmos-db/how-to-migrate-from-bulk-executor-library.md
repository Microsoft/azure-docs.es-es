---
title: Migración de la biblioteca Bulk Executor a la compatibilidad con la ejecución en bloque del SDK para .NET v3 de Azure Cosmos DB
description: Aprenda a migrar la aplicación del uso de la biblioteca Bulk Executor a la compatibilidad con la ejecución en bloque del SDK v3 de Azure Cosmos DB
author: ealsur
ms.service: cosmos-db
ms.topic: how-to
ms.date: 04/24/2020
ms.author: maquaran
ms.custom: devx-track-dotnet
ms.openlocfilehash: 8f573a3e851fe428c66066e36a913d6580cabd51
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89022486"
---
# <a name="migrate-from-the-bulk-executor-library-to-the-bulk-support-in-azure-cosmos-db-net-v3-sdk"></a>Migración de la biblioteca Bulk Executor a la compatibilidad con la ejecución en bloque del SDK para .NET v3 de Azure Cosmos DB

En este artículo se describen los pasos necesarios para migrar el código de una aplicación existente que usa la [biblioteca Bulk Executor de .NET](bulk-executor-dot-net.md) a la característica de [compatibilidad con la ejecución en bloque](tutorial-sql-api-dotnet-bulk-import.md) en la versión más reciente del SDK para .NET.

## <a name="enable-bulk-support"></a>Habilitar la compatibilidad con la ejecución en bloque

Habilite la compatibilidad con la ejecución en bloque en la instancia de `CosmosClient` mediante la configuración de [AllowBulkExecution](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.allowbulkexecution):

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/BulkExecutorMigration/Program.cs" ID="Initialization":::

## <a name="create-tasks-for-each-operation"></a>Creación de tareas para cada operación

La compatibilidad con la ejecución en bloque del SDK para .NET funciona mediante el uso de la [biblioteca TPL](https://docs.microsoft.com/dotnet/standard/parallel-programming/task-parallel-library-tpl) y las operaciones de agrupación que se producen simultáneamente. 

No hay ningún método único en el SDK que use su lista de documentos u operaciones como parámetro de entrada, sino que deberá crear una tarea para cada operación que quiera ejecutar de forma masiva. A continuación, simplemente, espere a que se completen.

Por ejemplo, si la entrada inicial es una lista de elementos en que cada elemento tiene el siguiente esquema:

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/BulkExecutorMigration/Program.cs" ID="Model":::

Si quiere realizar una importación masiva (una opción similar a usar BulkExecutor.BulkImportAsync), debe tener llamadas simultáneas a `CreateItemAsync`. Por ejemplo:

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/BulkExecutorMigration/Program.cs" ID="BulkImport":::

Si quiere realizar una *actualización* en bloque (una opción similar a usar [BulkExecutor.BulkUpdateAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor.bulkexecutor.bulkupdateasync)), necesita tener llamadas simultáneas al método `ReplaceItemAsync` después de actualizar el valor del elemento. Por ejemplo:

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/BulkExecutorMigration/Program.cs" ID="BulkUpdate":::

Y, si quiere realizar una *eliminación* en bloque (una opción similar a usar [BulkExecutor.BulkDeleteAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor.bulkexecutor.bulkdeleteasync)), necesita tener llamadas simultáneas a `DeleteItemAsync`, con `id` y la clave de partición de cada elemento. Por ejemplo:

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/BulkExecutorMigration/Program.cs" ID="BulkDelete":::

## <a name="capture-task-result-state"></a>Captura del estado del resultado de la tarea

En los ejemplos de código anteriores, hemos creado una lista de tareas simultáneas y ha llamado al método `CaptureOperationResponse` en cada una de esas tareas. Este método es una extensión que nos permite mantener un *esquema de respuesta similar* como BulkExecutor mediante la captura de los errores y el seguimiento del [uso de las unidades de solicitud](request-units.md).

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/BulkExecutorMigration/Program.cs" ID="CaptureOperationResult":::

Donde `OperationResponse` se declara como:

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/BulkExecutorMigration/Program.cs" ID="OperationResult":::

## <a name="execute-operations-concurrently"></a>Ejecución de operaciones simultáneas

Para realizar el seguimiento del ámbito de toda la lista de tareas, usamos esta clase auxiliar:

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/BulkExecutorMigration/Program.cs" ID="BulkOperationsHelper":::

El método `ExecuteAsync` esperará hasta que se completen todas las operaciones y se pueda usar de la siguiente manera:

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/BulkExecutorMigration/Program.cs" ID="WhenAll":::

## <a name="capture-statistics"></a>Captura de estadísticas

El código anterior espera hasta que se completan todas las operaciones y calcula las estadísticas necesarias. Estas estadísticas son similares a las del elemento [BulkImportResponse](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor.bulkimport.bulkimportresponse) de la biblioteca Bulk Executor.

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/BulkExecutorMigration/Program.cs" ID="ResponseType":::

`BulkOperationResponse` contiene:

1. El tiempo total que se tarda en procesar la lista de operaciones mediante la compatibilidad con la ejecución en bloque.
1. El número de operaciones completadas correctamente.
1. El total de unidades de la solicitud consumidas.
1. Si hay errores, muestra una lista de tuplas que contienen la excepción y el elemento asociado para fines de registro e identificación.

## <a name="retry-configuration"></a>Configuración de reintento

La biblioteca Bulk Executor tenía una [guía](bulk-executor-dot-net.md#bulk-import-data-to-an-azure-cosmos-account) que mencionaba que se debían establecer los valores de `MaxRetryWaitTimeInSeconds` y `MaxRetryAttemptsOnThrottledRequests` de [RetryOptions](https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.client.connectionpolicy.retryoptions) en `0` para delegar el control a la biblioteca.

Para la compatibilidad con la ejecución en bloque del SDK para .NET, no hay ningún comportamiento oculto. Puede configurar las opciones de reintento directamente mediante [CosmosClientOptions.MaxRetryAttemptsOnRateLimitedRequests](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.maxretryattemptsonratelimitedrequests) y [CosmosClientOptions.MaxRetryWaitTimeOnRateLimitedRequests](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.maxretrywaittimeonratelimitedrequests).

> [!NOTE]
> En los casos en los que las unidades de solicitud aprovisionadas están muy por debajo de las esperadas según la cantidad de datos, considere la posibilidad de definir valores altos para estas opciones. La operación en bloque tardará más, pero tendrá una mayor probabilidad de completarse correctamente debido a un valor de reintentos mayor.

## <a name="performance-improvements"></a>Mejoras en el rendimiento

Del mismo modo que con otras operaciones con el SDK para .NET, el uso de API de flujo da como resultado un mejor rendimiento y evita cualquier serialización innecesaria. 

El uso de API de flujo solo es posible si la naturaleza de los datos que usa coincide con la de un flujo de bytes (por ejemplo, flujos de archivos). En tales casos, el uso de los métodos `CreateItemStreamAsync`, `ReplaceItemStreamAsync` o `DeleteItemStreamAsync` y el trabajo con `ResponseMessage` (en lugar de `ItemResponse`) aumenta el rendimiento que se puede conseguir.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener más información acerca de las versiones del SDK para .NET, consulte el artículo [SDK de Azure Cosmos DB](sql-api-sdk-dotnet.md).
* Obtenga el [código fuente de migración](https://github.com/Azure/azure-cosmos-dotnet-v3/tree/master/Microsoft.Azure.Cosmos.Samples/Usage/BulkExecutorMigration) completo de GitHub.
* [Muestras adicionales de ejecución en bloque en GitHub](https://github.com/Azure/azure-cosmos-dotnet-v3/tree/master/Microsoft.Azure.Cosmos.Samples/Usage/BulkSupport)
