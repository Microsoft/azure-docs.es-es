---
title: Supervisión y depuración con métricas de Azure Cosmos DB
description: Use las métricas de Azure Cosmos DB para depurar problemas comunes y supervisar la base de datos.
author: kanshiG
ms.author: govindk
ms.reviewer: sngun
ms.service: cosmos-db
ms.topic: how-to
ms.date: 07/22/2020
ms.openlocfilehash: b8cf0425a4a40c4692f4c0c7fcf11dbb23019b23
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87132677"
---
# <a name="monitor-and-debug-with-metrics-in-azure-cosmos-db"></a>Supervisión y depuración con métricas de Azure Cosmos DB

Azure Cosmos DB proporciona métricas de rendimiento, almacenamiento, coherencia, disponibilidad y la latencia. En Azure Portal se proporciona una vista agregada de estas métricas. También puede ver las métricas de Azure Cosmos DB desde la API de Azure Monitor. Los valores de la dimensión para las métricas, como el nombre del contenedor, no distinguen entre mayúsculas y minúsculas. Por lo tanto, debe usar una comparación sin distinción entre mayúsculas y minúsculas al realizar comparaciones de cadenas en estos valores de dimensión. Para obtener información sobre cómo ver las métricas desde Azure Monitor, consulte el artículo [Obtener métricas desde Azure Monitor](cosmos-db-azure-monitor-metrics.md).

Este artículo le guía a través de casos de uso comunes y cómo se pueden utilizar las métricas de Azure Cosmos DB para analizar y depurar estos problemas. Las métricas se recopilan cada cinco minutos y se conservan durante siete días.

## <a name="view-metrics-from-azure-portal"></a>Visualización de las métricas desde Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. Abra el panel **Métricas**. De manera predeterminada, el panel de métricas muestra el almacenamiento, el índice y las métricas de unidades de solicitud para todas las bases de datos en su cuenta de Azure Cosmos. Puede filtrar estas métricas por base de datos, contenedor o una región. También puede filtrar las métricas en una granularidad de un momento determinado. Se proporcionan más detalles sobre las métricas de rendimiento, almacenamiento, disponibilidad, latencia y coherencia en pestañas independientes. 

   :::image type="content" source="./media/use-metrics/performance-metrics.png" alt-text="Métricas de rendimiento de Cosmos DB en Azure Portal":::

Las siguientes métricas están disponibles desde el panel **Métricas**: 

* **Métricas de rendimiento**: esta métrica muestra el número de solicitudes consumidas o con error (código de respuesta 429) debido a que se superó la capacidad de rendimiento o almacenamiento aprovisionada para el contenedor.

* **Métricas de almacenamiento**: esta métrica muestra el tamaño de los datos y el uso del índice.

* **Métricas de disponibilidad**: esta métrica muestra el porcentaje de solicitudes correctas realizadas en relación con el número total de solicitudes por hora. La tasa de éxito se define mediante los SLA de Azure Cosmos DB.

* **Métricas de latencia**: esta métrica muestra la latencia de lectura y escritura que observa Azure Cosmos DB en la región donde funciona su cuenta. Puede visualizar la latencia entre regiones para una cuenta de replicación geográfica. Esta métrica no representa la latencia de solicitud de un extremo a otro.

* **Métricas de coherencia**: esta métrica muestra cuán de eventual es la coherencia para el modelo de coherencia que elija. Para las cuentas de varias regiones, esta métrica muestra también la latencia de replicación entre las regiones que haya seleccionado.

* **Métricas del sistema**; esta métrica muestra cuántas solicitudes de metadatos procesa la partición principal. También ayuda a identificar las solicitudes limitadas.

En las siguientes secciones se explican escenarios comunes donde puede usar las métricas de Azure Cosmos DB. 

## <a name="understand-how-many-requests-are-succeeding-or-causing-errors"></a>Descripción de cuántas solicitudes se realizan correctamente o causan errores

Para empezar, vaya a [Azure Portal](https://portal.azure.com) y navegue hasta la hoja **Métricas**. En la hoja, busque el gráfico **Cantidad de solicitudes que superaron la capacidad durante 1 minuto. Este gráfico muestra las solicitudes totales minuto a minuto segmentadas por el código de estado. Para más información sobre los códigos de estado HTTP, consulte [HTTP Status Codes for Azure Cosmos DB](/rest/api/cosmos-db/http-status-codes-for-cosmosdb) (Códigos de estado HTTP para Azure Cosmos DB).

El código de estado de error más común es 429 (limitación de tasa/limitación). Este error significa que las solicitudes a Azure Cosmos DB superan el rendimiento aprovisionado. La solución más común para este problema consiste en [escalar verticalmente las RU](./set-throughput.md) para la colección dada.

:::image type="content" source="media/use-metrics/metrics-12.png" alt-text="Número de solicitudes por minuto":::

## <a name="determine-the-throughput-distribution-across-partitions"></a>Determinación de la distribución de rendimiento en las particiones

Tener una buena cardinalidad de las claves de partición es esencial para cualquier aplicación escalable. Para determinar la distribución de rendimiento de cualquier contenedor particionado dividido en particiones, vaya a la **hoja Métricas** en [Azure Portal](https://portal.azure.com). En la pestaña **Rendimiento**, se muestra el desglose de almacenamiento en el gráfico **Máximo de RU/segundo consumidas por cada partición física**. En el siguiente gráfico se ilustra un ejemplo de una distribución deficiente de los datos como lo evidencia la partición sesgada en el extremo izquierdo.

:::image type="content" source="media/use-metrics/metrics-17.png" alt-text="Una sola partición ve un uso intensivo":::

Una distribución de rendimiento desigual puede provocar particiones *activas*, lo que pueden dar lugar a solicitudes limitadas y la necesidad de volver a crear particiones. Para más información sobre la creación de particiones en Azure Cosmos DB, consulte [Partición y escalado en Azure Cosmos DB](./partition-data.md).

## <a name="determine-the-storage-distribution-across-partitions"></a>Determinación de la distribución de almacenamiento en las particiones

Tener una buena cardinalidad de la partición es esencial para cualquier aplicación escalable. Para determinar la distribución de almacenamiento de cualquier contenedor particionado dividido en particiones, vaya a la hoja Métricas en [Azure Portal](https://portal.azure.com). En la pestaña Almacenamiento, el desglose del almacenamiento se muestra en el gráfico Almacenamiento de datos e índice consumido por las principales claves de partición. En el siguiente gráfico se ilustra una distribución deficiente del almacenamiento de datos como lo evidencia la partición sesgada en el extremo izquierdo.

:::image type="content" source="media/use-metrics/metrics-07.png" alt-text="Ejemplo de distribución de datos deficiente":::

Puede determinar la causa raíz por la que la partición está sesgando la distribución haciendo clic en la partición en el gráfico.

:::image type="content" source="media/use-metrics/metrics-05.png" alt-text="Clave de partición que sesga la distribución":::

Después de identificar qué clave de partición está causando el sesgo en la distribución, puede que tenga que volver a crear particiones en el contenedor con una clave de partición más distribuida. Para más información sobre la creación de particiones en Azure Cosmos DB, consulte [Partición y escalado en Azure Cosmos DB](./partition-data.md).

## <a name="compare-data-size-against-index-size"></a>Comparación del tamaño de los datos con el tamaño de índice

En Azure Cosmos DB, el almacenamiento total consumido es la combinación del tamaño de los datos y del tamaño del índice. Normalmente, el tamaño del índice es una fracción del tamaño de los datos. En la hoja Métricas de [Azure Portal](https://portal.azure.com), la pestaña Almacenamiento muestra el desglose de consumo de almacenamiento en función de los datos y el índice.

```csharp
// Measure the document size usage (which includes the index size)  
ResourceResponse<DocumentCollection> collectionInfo = await client.ReadDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri("db", "coll"));
 Console.WriteLine("Document size quota: {0}, usage: {1}", collectionInfo.DocumentQuota, collectionInfo.DocumentUsage);
```

Si desea conservar el espacio de índice, puede ajustar la [directiva de indexación](index-policy.md).

## <a name="debug-why-queries-are-running-slow"></a>Depuración de por qué mis consultas se ejecutan lentamente

En los SDK de la API de SQL, Azure Cosmos DB proporciona estadísticas de ejecución de consultas.

```csharp
IDocumentQuery<dynamic> query = client.CreateDocumentQuery(
 UriFactory.CreateDocumentCollectionUri(DatabaseName, CollectionName),
 "SELECT * FROM c WHERE c.city = 'Seattle'",
 new FeedOptions
 {
 PopulateQueryMetrics = true,
 MaxItemCount = -1,
 MaxDegreeOfParallelism = -1,
 EnableCrossPartitionQuery = true
 }).AsDocumentQuery();
FeedResponse<dynamic> result = await query.ExecuteNextAsync();

// Returns metrics by partition key range Id
IReadOnlyDictionary<string, QueryMetrics> metrics = result.QueryMetrics;
```

*QueryMetrics* proporciona información detallada sobre cuánto tardó cada componente de la consulta en la ejecución. La causa principal más común para consultas de larga ejecución son los análisis, lo que significa que la consulta no pudo aprovechar los índices. Este problema puede resolverse con una mejor condición de filtro.

## <a name="next-steps"></a>Pasos siguientes

Ha aprendido a supervisar y depurar problemas con las métricas proporcionadas en Azure Portal. Si quiere obtener más información acerca de cómo mejorar el rendimiento de bases de datos, lea los artículos siguientes:

* Para obtener información sobre cómo ver las métricas desde Azure Monitor, consulte el artículo [Obtener métricas desde Azure Monitor](cosmos-db-azure-monitor-metrics.md). 
* [Pruebas de escala y rendimiento con Azure Cosmos DB](performance-testing.md)
* [Sugerencias de rendimiento para Azure Cosmos DB](performance-tips.md)
