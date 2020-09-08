---
title: Supervisión del estado y los resultados del indizador
titleSuffix: Azure Cognitive Search
description: Supervise el estado, el progreso y los resultados de los indizadores de Azure Cognitive Search en Azure Portal mediante la API REST o el SDK de .NET.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.devlang: rest-api
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 07/12/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 649611b2e378cd43286b193c6d40b03b743905cd
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89000080"
---
# <a name="how-to-monitor-azure-cognitive-search-indexer-status-and-results"></a>Supervisión del estado y los resultados del indizador de Azure Cognitive Search

Azure Cognitive Search proporciona información sobre el estado y la supervisión de las ejecuciones actuales e históricas de todos los indizadores.

La supervisión de los indizadores es útil si quiere:

* Llevar un seguimiento del progreso de un indizador durante una ejecución en curso.
* Revisar los resultados de una ejecución en curso o anterior del indizador.
* Identificar errores del indizador de nivel superior, así como errores o advertencias sobre documentos individuales que se están indexando.

## <a name="get-status-and-history"></a>Obtención del estado e historial

Puede acceder a la información de supervisión del indizador de varias maneras, incluidas las siguientes:

* En [Azure Portal](#portal)
* Mediante la [API de REST](#restapi)
* Mediante el [SDK de .NET](#dotnetsdk)

La información disponible sobre la supervisión del indizador incluye todo lo siguiente (aunque los formatos de los datos difieren en función del método de acceso usado):

* Información de estado sobre el propio indizador.
* Información sobre la última ejecución del indizador, incluido su estado, la hora de inicio y finalización, y errores y advertencias detallados.
* Una lista de ejecuciones históricas del indizador y sus estados, resultados, errores y advertencias.

Los indizadores que procesan grandes volúmenes de datos pueden tardar mucho en ejecutarse. Por ejemplo, los indizadores que tratan millones de documentos de origen pueden ejecutarse durante 24 horas y, después, reiniciarse casi de inmediato. El estado de los indizadores de gran volumen podría indicar siempre **En curso** en el portal. Incluso cuando se está ejecutando un indizador, los detalles sobre el progreso actual y las ejecuciones anteriores están disponibles.

<a name="portal"></a>

## <a name="monitor-using-the-portal"></a>Supervisión del uso del portal

Puede consultar el estado actual de todos los indizadores en la lista **Indizadores**, en la página de información general del servicio de búsqueda.

   ![Lista de indizadores](media/search-monitor-indexers/indexers-list.png "Lista de indizadores")

Cuando se está ejecutando un indizador, el estado de la lista es **En curso** y el valor **Documentos correctos** indica el número de documentos procesados hasta el momento. El portal puede tardar unos minutos en actualizar los valores de estado del indizador y el número de documentos.

Los indizadores cuya última ejecución haya sido correcta mostrarán el estado **Correcto**. Una ejecución del indizador puede ser correcta incluso si los documentos individuales tienen errores, siempre y cuando el número de errores sea inferior al valor **Máx. de elementos con error** del indizador.

Si la última ejecución finaliza con un error, se muestra el estado **Erróneo**. El estado **Restablecer** indica que se ha restablecido el estado de seguimiento de cambios del indizador.

Haga clic en un indizador de la lista para ver más detalles sobre sus ejecuciones actuales y recientes.

   ![Historial de ejecución y resumen del indizador](media/search-monitor-indexers/indexer-summary.png "Historial de ejecución y resumen del indizador")

En el gráfico **Resumen del indizador** se muestra el número de documentos procesados en las ejecuciones más recientes.

En la lista **Detalles de la ejecución** aparecen como máximo los 50 resultados de las ejecuciones más recientes.

Haga clic en un resultado de la lista para ver los detalles de una ejecución en concreto. Se incluyen las horas de inicio y finalización, así como los errores y las advertencias que se han producido.

   ![Detalles de ejecución del indizador](media/search-monitor-indexers/indexer-execution.png "Detalles de ejecución del indizador")

Si se produjeron problemas específicos del documento durante la ejecución, se mostrarán en el campo Errores o advertencias.

   ![Detalles del indizador con errores](media/search-monitor-indexers/indexer-execution-error.png "Detalles del indizador con errores")

Las advertencias son comunes con algunos tipos de indizadores y no siempre indican un problema. Por ejemplo, los indizadores que usan Cognitive Services pueden notificar advertencias cuando las imágenes o los archivos PDF no contienen texto para procesar.

Para obtener más información sobre cómo investigar los errores y las advertencias del indizador, vea [Solución de problemas comunes con el indizador en Azure Search](search-indexer-troubleshooting.md).

<a name="restapi"></a>

## <a name="monitor-using-rest-apis"></a>Supervisión mediante las API REST

Puede recuperar el estado y el historial de ejecución de un indizador con el [comando Obtener estado del indizador](/rest/api/searchservice/get-indexer-status):

```http
GET https://[service name].search.windows.net/indexers/[indexer name]/status?api-version=2020-06-30
api-key: [Search service admin key]
```

La respuesta contiene el estado general del indexador, la última vez que se invocó al indexador (o en curso) y el historial de las recientes invocaciones al indexador.

```output
{
    "status":"running",
    "lastResult": {
        "status":"success",
        "errorMessage":null,
        "startTime":"2018-11-26T03:37:18.853Z",
        "endTime":"2018-11-26T03:37:19.012Z",
        "errors":[],
        "itemsProcessed":11,
        "itemsFailed":0,
        "initialTrackingState":null,
        "finalTrackingState":null
     },
    "executionHistory":[ {
        "status":"success",
         "errorMessage":null,
        "startTime":"2018-11-26T03:37:18.853Z",
        "endTime":"2018-11-26T03:37:19.012Z",
        "errors":[],
        "itemsProcessed":11,
        "itemsFailed":0,
        "initialTrackingState":null,
        "finalTrackingState":null
    }]
}
```

El historial de ejecución contiene como máximo las 50 ejecuciones más recientes, dispuestas en orden cronológico inverso (la más reciente en primer lugar).

Tenga en cuenta que hay dos valores de estado diferentes. El estado de nivel superior es para el propio indizador. El estado del indizador **En ejecución** significa que el indizador se ha configurado correctamente y está disponible para su ejecución, pero no que se esté ejecutando actualmente.

Cada ejecución del indizador también tiene su propio estado, que indica si la ejecución está en curso (**En ejecución**) o se ha completado con el estado **correcto**, **transientFailure** o **persistentFailure**. 

Cuando se restablece un indizador para actualizar su estado de seguimiento de cambios, se agrega una entrada independiente en el historial de ejecución con el estado **Restablecer**.

Para obtener más información sobre los códigos de estado y los datos de supervisión del indizador, consulte [GetIndexerStatus](/rest/api/searchservice/get-indexer-status).

<a name="dotnetsdk"></a>

## <a name="monitor-using-the-net-sdk"></a>Supervisión mediante el SDK de .NET

La programación de un indizador se puede definir mediante el SDK de .NET de Azure Cognitive Search. Para ello, incluya la propiedad **schedule** al crear o actualizar cualquier indexador.

En el siguiente ejemplo de C# se escribe información sobre el estado de un indizador y los resultados de su ejecución más reciente (o en curso) en la consola.

```csharp
static void CheckIndexerStatus(Indexer indexer, SearchServiceClient searchService)
{
    try
    {
        IndexerExecutionInfo execInfo = searchService.Indexers.GetStatus(indexer.Name);

        Console.WriteLine("Indexer has run {0} times.", execInfo.ExecutionHistory.Count);
        Console.WriteLine("Indexer Status: " + execInfo.Status.ToString());

        IndexerExecutionResult result = execInfo.LastResult;

        Console.WriteLine("Latest run");
        Console.WriteLine("  Run Status: {0}", result.Status.ToString());
        Console.WriteLine("  Total Documents: {0}, Failed: {1}", result.ItemCount, result.FailedItemCount);

        TimeSpan elapsed = result.EndTime.Value - result.StartTime.Value;
        Console.WriteLine("  StartTime: {0:T}, EndTime: {1:T}, Elapsed: {2:t}", result.StartTime.Value, result.EndTime.Value, elapsed);

        string errorMsg = (result.ErrorMessage == null) ? "none" : result.ErrorMessage;
        Console.WriteLine("  ErrorMessage: {0}", errorMsg);
        Console.WriteLine("  Document Errors: {0}, Warnings: {1}\n", result.Errors.Count, result.Warnings.Count);
    }
    catch (Exception e)
    {
        // Handle exception
    }
}
```

La salida de la consola tendrá un aspecto similar al siguiente:

```output
Indexer has run 18 times.
Indexer Status: Running
Latest run
  Run Status: Success
  Total Documents: 7, Failed: 0
  StartTime: 10:02:46 PM, EndTime: 10:02:47 PM, Elapsed: 00:00:01.0990000
  ErrorMessage: none
  Document Errors: 0, Warnings: 0
```

Tenga en cuenta que hay dos valores de estado diferentes. El estado de nivel superior es el del propio indizador. El estado del indizador **En ejecución** significa que el indizador se ha configurado correctamente y está disponible para su ejecución, pero no que se esté ejecutando actualmente.

Cada ejecución del indizador también tiene su propio estado, que indica si la ejecución está en curso (**En ejecución**) o se ha completado con el estado **Correcto** o **TransientError**. 

Cuando se restablece un indizador para actualizar su estado de seguimiento de cambios, se agrega una entrada independiente en el historial con el estado **Restablecer**.

Para obtener más detalles sobre los códigos de estado y la información de supervisión del indizador, consulte [GetIndexerStatus](/rest/api/searchservice/get-indexer-status) en la API de REST.

Para recuperar información sobre errores o advertencias específicos del documento, enumere las listas `IndexerExecutionResult.Errors` y `IndexerExecutionResult.Warnings`.

Para obtener más información sobre las clases del SDK de .NET que se usan para supervisar los indizadores, vea [IndexerExecutionInfo](/dotnet/api/microsoft.azure.search.models.indexerexecutioninfo?view=azure-dotnet) e [IndexerExecutionResult](/dotnet/api/microsoft.azure.search.models.indexerexecutionresult?view=azure-dotnet).