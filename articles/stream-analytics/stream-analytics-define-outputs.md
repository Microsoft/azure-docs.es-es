---
title: Salidas de Azure Stream Analytics
description: En este artículo se describen las opciones de salida de datos disponibles para Azure Stream Analytics.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 08/25/2020
ms.openlocfilehash: 6576ac53f43a8e176b3d166d5218b0ba9934b856
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90907162"
---
# <a name="outputs-from-azure-stream-analytics"></a>Salidas de Azure Stream Analytics

Un trabajo de Azure Stream Analytics consta de una entrada, una consulta y una salida. Hay varios tipos de salida a los que puede enviar datos transformados. En este artículo se enumeran las salidas de Stream Analytics admitidas. Al diseñar la consulta de Stream Analytics, haga referencia al nombre de la salida mediante la [cláusula INTO](https://docs.microsoft.com/stream-analytics-query/into-azure-stream-analytics). Puede usar una única salida por trabajo, o varias salidas por trabajo de streaming (si las necesita); para ello, agregue varias cláusulas INTO en la consulta.

Para crear, editar y probar las salidas de trabajos de Stream Analytics, puede usar [Azure Portal](stream-analytics-quick-create-portal.md#configure-job-output), [Azure PowerShell](stream-analytics-quick-create-powershell.md#configure-output-to-the-job), la [API de .NET](https://docs.microsoft.com/dotnet/api/microsoft.azure.management.streamanalytics.ioutputsoperations?view=azure-dotnet), la [API REST](https://docs.microsoft.com/rest/api/streamanalytics/stream-analytics-output) y [Visual Studio](stream-analytics-quick-create-vs.md).

Algunos tipos de salida admiten la [creación de particiones](#partitioning), y los [tamaños de los lotes de salida](#output-batch-size) varían para optimizar el rendimiento. En la siguiente tabla se muestran las características que se admiten para cada tipo de salida:

| Tipo de salida | Creación de particiones | Seguridad | 
|-------------|--------------|----------|
|[Azure Data Lake Storage Gen 1](azure-data-lake-storage-gen1-output.md)|Sí|Usuario de Azure Active Directory </br> MSI|
|[Azure SQL Database](sql-database-output.md)|Sí, debe habilitarse.|Autenticación de usuario de SQL </br> MSI (versión preliminar)|
|[Azure Synapse Analytics](azure-synapse-analytics-output.md)|No|Autenticación de usuario de SQL|
|[Blob Storage y Azure Data Lake Gen 2](blob-storage-azure-data-lake-gen2-output.md)|Sí|MSI </br> Clave de acceso|
|[Azure Event Hubs](event-hubs-output.md)|Sí|Clave de acceso|
|[Power BI](power-bi-output.md)|No|Usuario de Azure Active Directory </br> MSI|
|[Azure Table Storage](table-storage-output.md)|Sí|Clave de cuenta|
|[Colas de Azure Service Bus](service-bus-queues-output.md)|Sí|Clave de acceso|
|[Temas de Azure Service Bus](service-bus-topics-output.md)|Sí|Clave de acceso|
|[Azure Cosmos DB](azure-cosmos-db-output.md)|Sí|Clave de acceso|
|[Funciones de Azure](azure-functions-output.md)|Sí|Clave de acceso|

## <a name="partitioning"></a>Creación de particiones

Stream Analytics admite particiones para todas las salidas, excepto para Power BI. Para más información sobre las claves de partición y el número de escritores de salida, consulte el artículo sobre el tipo de salida específico que le interesa. Todos los artículos de salidas están vinculados en la sección anterior.  

Se puede controlar el número de escritores de salida mediante una cláusula `INTO <partition count>` (consulte [INTO](https://docs.microsoft.com/stream-analytics-query/into-azure-stream-analytics#into-shard-count)) de la consulta, lo cual puede ser útil para lograr la topología de trabajo deseada. Si el adaptador de salida no tiene particiones, la falta de datos de una partición de entrada provocará un retraso hasta la cantidad de tiempo de llegada tardía. En tales casos, la salida se combina en un único sistema de escritura, lo que puede provocar cuellos de botella en la canalización. Para obtener más información sobre la directiva de llegadas tardías, consulte [Puntos a tener en cuenta sobre el orden de eventos de Azure Stream Analytics](stream-analytics-out-of-order-and-late-events.md).

## <a name="output-batch-size"></a>Tamaño de lote de salida

Todas las salidas admiten el procesamiento por lotes, pero solo algunas admiten el ajuste de tamaño de los lotes. Azure Stream Analytics usa lotes de tamaño variable para procesar eventos y escribir en las salidas. Normalmente, el motor de Stream Analytics no escribe un mensaje a la vez y utiliza lotes para mejorar la eficacia. Cuando la velocidad de los eventos entrantes y salientes es alta, Stream Analytics utiliza lotes más grandes. Cuando la tasa de salida es baja, utiliza lotes más pequeños para mantener baja la latencia.

## <a name="parquet-output-batching-window-properties"></a>Propiedades de la ventana de procesamiento por lotes con salida de Parquet

Cuando se usa la implementación con plantilla de Azure Resource Manager o la API REST, las dos propiedades de la ventana de procesamiento por lotes son:

1. *timeWindow*

   Tiempo de espera máximo por lote. El valor debe ser una cadena de intervalo de tiempo. Por ejemplo, "00:02:00" para dos minutos. Después de este tiempo, el lote se escribe en la salida aunque no se cumpla el requisito de filas mínimas. El valor predeterminado es 1 minuto y el máximo permitido son 2 horas. Si la salida del blob tiene una frecuencia de patrón de ruta de acceso, el tiempo de espera no puede ser mayor que el intervalo de tiempo de la partición.

2. *sizeWindow*

   Número mínimo de filas por lote. En el caso de Parquet, cada lote crea un archivo. El valor predeterminado actual es 2000 filas y el máximo permitido es 10 000 filas.

Estas propiedades de ventana de procesamiento por lotes solo se admiten en la versión de API **2017-04-01-Preview**. A continuación se muestra un ejemplo de la carga de JSON para una llamada API REST:

```json
"type": "stream",
      "serialization": {
        "type": "Parquet",
        "properties": {}
      },
      "timeWindow": "00:02:00",
      "sizeWindow": "2000",
      "datasource": {
        "type": "Microsoft.Storage/Blob",
        "properties": {
          "storageAccounts" : [
          {
            "accountName": "{accountName}",
            "accountKey": "{accountKey}",
          }
          ],
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
>
> [Inicio rápido: Creación de un trabajo de Stream Analytics mediante Azure Portal](stream-analytics-quick-create-portal.md)

<!--Link references-->
[stream.analytics.developer.guide]: ../stream-analytics-developer-guide.md
[stream.analytics.scale.jobs]: stream-analytics-scale-jobs.md
[stream.analytics.introduction]: stream-analytics-introduction.md
[stream.analytics.get.started]: stream-analytics-real-time-fraud-detection.md
[stream.analytics.query.language.reference]: https://go.microsoft.com/fwlink/?LinkID=513299
[stream.analytics.rest.api.reference]: https://go.microsoft.com/fwlink/?LinkId=517301
