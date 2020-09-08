---
title: Optimización de los costos de almacenamiento de Azure Cosmos DB
description: En este artículo se explica cómo administrar los costos de almacenamiento de los datos almacenados en Azure Cosmos DB.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 05/21/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: dca046df68b10853752b0de65c48c2b8f83afb31
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89020905"
---
# <a name="optimize-storage-cost-in-azure-cosmos-db"></a>Optimización de los costos de almacenamiento de Azure Cosmos DB

Azure Cosmos DB ofrece almacenamiento y rendimiento ilimitados. A diferencia del rendimiento, que tiene que aprovisionar o configurar en los contenedores o bases de datos de Azure Cosmos, el almacenamiento se factura en base al consumo. Solo se le cobrará por el almacenamiento lógico que consume y no tiene que reservar ningún almacenamiento por adelantado. El almacenamiento se escala vertical y horizontalmente de manera automática en función de los datos que se agregan o eliminan en un contenedor de Azure Cosmos.

## <a name="storage-cost"></a>Coste del almacenamiento

El almacenamiento se factura con la unidad de GB. El almacenamiento local respaldado por SSD lo utilizan los datos y la indexación. El almacenamiento total utilizado es igual al almacenamiento requerido por los datos e índices utilizados en todas las regiones donde se utiliza Azure Cosmos DB. Si se replica globalmente una cuenta de Azure Cosmos en tres regiones, pagará por el costo total de almacenamiento en cada una de esas tres regiones. Para calcular los requisitos de almacenamiento, consulte la herramienta de [planeamiento de capacidad](https://www.documentdb.com/capacityplanner). El costo de almacenamiento de Azure Cosmos DB es $0,25 GB/mes, consulte en la [página de precios](https://azure.microsoft.com/pricing/details/cosmos-db/) las actualizaciones más recientes. Puede configurar alertas para determinar el almacenamiento utilizado por el contenedor de Azure Cosmos; para supervisar el almacenamiento, consulte el artículo [Supervisión de Azure Cosmos DB](monitor-accounts.md)).

## <a name="optimize-cost-with-item-size"></a>Optimización de los costos con el tamaño del elemento

Azure Cosmos DB espera que el tamaño del elemento sea de 2 MB o menos para un rendimiento óptimo y ventajas de costo. Si necesita que algún elemento almacene más de 2 MB de datos, considere la posibilidad de volver a diseñar el esquema del elemento. En el raro caso de que no pueda volver a diseñar el esquema, puede dividir el elemento en subelementos y vincularlos lógicamente con un identificador común. Todas las características de Azure Cosmos DB funcionan de manera coherente mediante la delimitación de ese identificador lógico.

## <a name="optimize-cost-with-indexing"></a>Optimización del costo con indexación

De forma predeterminada, los datos se indexan automáticamente, lo que puede aumentar el total de almacenamiento consumido. Sin embargo, puede aplicar directivas de índice personalizadas para reducir esta sobrecarga. La indexación automática que no se ha ajustado mediante la directiva es de aproximadamente del 10 a 20 % del tamaño del elemento. Al quitar o personalizar las directivas de índice, no pagará ningún costo adicional por las operaciones de escritura y no necesitará capacidad de rendimiento adicional. Consulte [Indexación en Azure Cosmos DB](indexing-policies.md) para configurar directivas de indexación personalizadas. Si ha trabajado antes con bases de datos relacionales, puede pensar que "indexar todo" significa duplicar el almacenamiento o algo más. Sin embargo, en Azure Cosmos DB, en el caso medio, es mucho menor. En Azure Cosmos DB, la sobrecarga de almacenamiento del índice es típicamente baja (10-20 %) incluso con la indexación automática, porque está diseñada para un espacio de almacenamiento reducido. Mediante la administración de la directiva de indexación, puede controlar el equilibrio de la superficie del índice y del rendimiento de la consulta de una manera más precisa.

## <a name="optimize-cost-with-time-to-live-and-change-feed"></a>Optimización del costo con el período de vida y la fuente de cambios

Cuando ya no necesite los datos, puede eliminarlos de su cuenta de Azure Cosmos mediante el [período de vida](time-to-live.md), [fuente de cambios](change-feed.md) o puede migrar los datos antiguos a otro almacén de datos como el almacenamiento de blobs de Azure o el almacenamiento de datos de Azure. Mediante el período de vida o TTL, Azure Cosmos DB proporciona la capacidad de eliminar automáticamente elementos de un contenedor después de un determinado período de tiempo. De forma predeterminada, puede establecer el período de vida en el nivel de contenedor e invalidar el valor en cada elemento. Después de establecer el período de vida en el nivel de contenedor o de elemento, Azure Cosmos DB eliminará automáticamente estos elementos cuando haya pasado el período de tiempo seleccionado tras la hora de la última modificación. Al utilizar la fuente de cambios, puede migrar datos a otro contenedor en Azure Cosmos DB o a un almacén de datos externo. La migración toma cero tiempo de inactividad y cuando termine de migrar, puede eliminar o configurar el tiempo de vida para eliminar el contenedor de origen de Azure Cosmos.

## <a name="optimize-cost-with-rich-media-data-types"></a>Optimización del costo con tipos de datos de medios enriquecidos 

Si desea almacenar tipos de medios enriquecidos, por ejemplo, vídeos o imágenes, tiene varias opciones en Azure Cosmos DB. Una opción es almacenar estos tipos de medios enriquecidos como elementos de Azure Cosmos. Hay un límite de 2 MB por elemento, y puede evitarlo con el encadenamiento del elemento de datos en varios subelementos. O puede almacenarlos en Azure Blob Storage y usar los metadatos para hacer referencia a ellos desde sus elementos de Azure Cosmos. Hay una serie de ventajas y desventajas de este enfoque. El primer enfoque le proporciona el mejor rendimiento en términos de latencia, SLA de rendimiento y funcionalidades de distribución global llave en mano para los tipos de datos de medios enriquecidos, además de los elementos habituales de Azure Cosmos. Sin embargo, el soporte técnico está disponible a un precio más alto. Al almacenar los medios en Azure Blob Storage, podría reducir sus costos generales. Si la latencia es crítica, puede usar Premium Storage para archivos de medios enriquecidos a los que se hace referencia desde los elementos de Azure Cosmos. Esto se integra de forma nativa con la red CDN para servir imágenes desde el servidor perimetral a un coste menor para evitar la restricción geográfica. El inconveniente de este escenario es que tiene que tratar con dos servicios: Azure Cosmos DB y Azure Blob Storage, lo que puede aumentar los costos operativos. 

## <a name="check-storage-consumed"></a>Comprobación del almacenamiento consumido

Para comprobar el consumo de almacenamiento de un contenedor de Azure Cosmos, puede ejecutar una solicitud HEAD o GET en el contenedor, e inspeccionar los encabezados `x-ms-request-quota` y `x-ms-request-usage`. Además, cuando trabaje con SDK de .NET, puede utilizar las propiedades [DocumentSizeQuota](https://docs.microsoft.com/previous-versions/azure/dn850325(v%3Dazure.100)), y [DocumentSizeUsage](https://msdn.microsoft.com/library/azure/dn850324.aspx) para consumir el almacenamiento.

## <a name="using-sdk"></a>Uso de SDK

```csharp
// Measure the item size usage (which includes the index size)
ResourceResponse<DocumentCollection> collectionInfo = await client.ReadDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri("db", "coll"));   

Console.WriteLine("Item size quota: {0}, usage: {1}", collectionInfo.DocumentQuota, collectionInfo.DocumentUsage);
```

## <a name="next-steps"></a>Pasos siguientes

A continuación, puede seguir obteniendo más información sobre la optimización de costos en Azure Cosmos DB con los siguientes artículos:

* Obtenga más información sobre la [optimización para desarrollo y pruebas](optimize-dev-test.md).
* Obtenga más información sobre [la factura de Azure Cosmos DB](understand-your-bill.md).
* Obtenga más información sobre la [optimización del costo de la capacidad del rendimiento](optimize-cost-throughput.md).
* Obtenga más información sobre la [optimización del costo de la lectura y la escritura](optimize-cost-reads-writes.md).
* Obtenga más información sobre la [optimización del costo de las consultas](optimize-cost-queries.md).
* Obtenga más información sobre la [optimización del costo de las cuentas de Azure Cosmos de varias regiones](optimize-cost-regions.md).

