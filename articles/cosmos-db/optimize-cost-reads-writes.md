---
title: Optimización del costo de lecturas y escrituras en Azure Cosmos DB
description: En este artículo se explica cómo reducir los costos de Azure Cosmos DB al realizar las operaciones de lectura y escritura en los datos.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 07/24/2020
ms.openlocfilehash: 38084bf30df2a597e7a7bc46ba4c52cf371c3c7e
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87318256"
---
# <a name="optimize-reads-and-writes-cost-in-azure-cosmos-db"></a>Optimización del costo de lecturas y escrituras en Azure Cosmos DB

En este artículo se describe cómo se calcula el costo necesario para leer y escribir datos desde Azure Cosmos DB. Las operaciones de lectura incluyen [lecturas y consultas puntuales](sql-query-getting-started.md). Las operaciones de escritura incluyen la inserción, el reemplazo, la eliminación y la inserción de elementos.  

## <a name="cost-of-reads-and-writes"></a>Costo de lecturas y escrituras

Azure Cosmos DB garantiza un rendimiento predecible en cuanto a rendimiento y latencia con un modelo de rendimiento aprovisionado. El rendimiento aprovisionado se representa en términos de [unidades de solicitud](request-units.md) por segundo o RU/s. Una unidad de solicitud (RU) es una abstracción lógica a través de los recursos de proceso, como CPU, memoria, E/S, etc., que son necesarios para realizar una solicitud. El rendimiento aprovisionado (RU) se reserva y está dedicado a su contenedor o base de datos para ofrecer un rendimiento y latencia predecibles. El rendimiento aprovisionado permite a Azure Cosmos DB ofrecer un rendimiento coherente y predecible, baja latencia garantizada y alta disponibilidad a cualquier escala. Las unidades de solicitud representan la moneda normalizada que simplifica el razonamiento sobre cuántos recursos necesita una aplicación.

No tiene que pensar en diferenciar unidades de solicitud entre lecturas y escrituras. El modelo de moneda unificado de unidades de solicitud crea eficiencias para usar indistintamente la misma capacidad de rendimiento para lecturas y escrituras. En la tabla siguiente se muestra el costo de lecturas y escrituras puntuales en términos de RU/s para los elementos con un tamaño de 1 KB y de 100 KB.

|**Tamaño del elemento**  |**Costo de una lectura puntual** |**Costo de una escritura**|
|---------|---------|---------|
|1 KB |1 RU |5 RU |
|100 KB |10 RU |50 RU |

La lectura puntual de un elemento de 1 KB de tamaño cuesta 1 RU. Escribir un elemento de 1 KB cuesta 5 RU. Los costos de lectura y escritura corresponden cuando se usa el [nivel de coherencia](consistency-levels.md) de la sesión predeterminada.  Las consideraciones acerca de las RU incluyen: el tamaño del elemento, el recuento de propiedades, la coherencia de los datos, las propiedades indexadas, la indexación y los patrones de consulta.

Las [lecturas puntuales](sql-query-getting-started.md) cuestan significativamente menos RU que las consultas. Al contrario que las consultas, las lecturas puntuales no necesitan el motor de consultas para acceder a los datos y ahorran en RU. El cargo en RU por las consultas depende de su complejidad y el número de elementos que tenga que cargar el motor de consultas.

## <a name="optimize-the-cost-of-writes-and-reads"></a>Optimización del costo de lecturas y escrituras

Al realizar operaciones de escritura, debe aprovisionar capacidad suficiente para admitir el número de escrituras necesarias por segundo. Puede aumentar el rendimiento aprovisionado mediante el SDK, el portal, la CLI antes de realizar las escrituras y, luego, reducir el rendimiento una vez completadas las escrituras. El rendimiento durante el período de escritura es el rendimiento mínimo necesario para los datos especificados, además del rendimiento necesario para la carga de trabajo de insert, suponiendo que no hay otras cargas de trabajo en ejecución.

Si se ejecutan otras cargas de trabajo simultáneamente, por ejemplo, consultar/leer/actualizar/eliminar, también debe agregar las unidades de solicitud adicionales necesarias para esas operaciones. Si las operaciones de escritura tienen una velocidad limitada, puede personalizar la directiva de retroceso/reintento mediante los SDK de Azure Cosmos DB. Por ejemplo, puede aumentar la carga hasta que una pequeña tasa de solicitudes obtenga una velocidad limitada. Si se produce el límite de velocidad, la aplicación cliente debe retroceder en las solicitudes con limitación de velocidad para el intervalo de reintento especificado. Antes de reintentar las escrituras, debe tener una cantidad mínima de tiempo entre reintentos. Se incluye compatibilidad con la directiva de reintentos en los SDK de SQL. NET, Java, Node.js y Python, y todas las versiones compatibles de los SDK de .NET Core.

También puede insertar masivamente datos en Azure Cosmos DB o copiar datos desde cualquier almacén de datos de origen compatible a Azure Cosmos DB mediante [Azure Data Factory](../data-factory/connector-azure-cosmos-db.md). Data Factory se integra de manera nativa con la API Bulk de Azure Cosmos DB para proporcionar el mejor rendimiento de escritura de datos.

## <a name="next-steps"></a>Pasos siguientes

A continuación, puede seguir obteniendo más información sobre la optimización de costos en Azure Cosmos DB con los siguientes artículos:

* Obtenga más información sobre la [optimización para desarrollo y pruebas](optimize-dev-test.md).
* Obtenga más información sobre [la factura de Azure Cosmos DB](understand-your-bill.md).
* Obtenga más información sobre la [optimización del costo de la capacidad del rendimiento](optimize-cost-throughput.md).
* Obtenga más información sobre la [optimización del costo del almacenamiento](optimize-cost-storage.md).
* Obtenga más información sobre la [optimización del costo de las consultas](optimize-cost-queries.md).
* Obtenga más información sobre la [optimización del costo de las cuentas de Azure Cosmos de varias regiones](optimize-cost-regions.md).
