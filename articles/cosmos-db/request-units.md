---
title: Rendimiento y unidades de solicitud en Azure Cosmos DB
description: Obtenga información sobre cómo especificar y estimar los requisitos de las unidades de solicitud en Azure Cosmos DB.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 07/24/2020
ms.openlocfilehash: f1f203d17de9fb0fc9fe8bb0f6de80fe2b93ba8b
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87327810"
---
# <a name="request-units-in-azure-cosmos-db"></a>Unidades de solicitud en Azure Cosmos DB

Con Azure Cosmos DB, paga por el rendimiento que aprovisiona y el almacenamiento que consume cada hora. Se debe aprovisionar el rendimiento para garantizar que haya disponible suficientes recursos del sistema para la base de datos de Azure Cosmos en todo momento. Necesita los recursos suficientes para cumplir o superar los [acuerdos de nivel de servicio de Azure Cosmos DB](https://azure.microsoft.com/support/legal/sla/cosmos-db/v1_2/).

Azure Cosmos DB admite varias API, como SQL, MongoDB, Cassandra, Gremlin y Table. Cada API tiene su propio conjunto de operaciones de base de datos. Estas abarcan desde sencillas lecturas y escrituras de punto hasta consultas complejas. Cada operación de base de datos consume recursos del sistema en función de la complejidad de la operación.

Azure Cosmos DB se encarga de normalizar el costo de todas las operaciones de base de datos y se expresa en términos de *unidades de solicitud* (RU en su forma abreviada). RU/segundo se puede considerar como la moneda del rendimiento. RU/s es una moneda basada en la velocidad, que abstrae los recursos del sistema, como CPU, IOPS y memoria, necesarios para realizar las operaciones de base de datos compatibles con Azure Cosmos DB. Se requiere un mínimo de 10 RU/s para almacenar cada GB de datos.

El costo de hacer una lectura puntual de un elemento de 1 KB es una unidad de solicitud (o 1 RU). A todas las demás operaciones de base de datos se les asigna de forma similar un costo en términos de unidades de solicitud. Con independencia de qué API utilice para interactuar con el contenedor de Azure Cosmos, los costos siempre se miden por RU. Si la operación de base de datos es una escritura, lectura puntual o consulta, los costos siempre se miden en RU.

En la siguiente imagen se muestra el concepto de unidades de solicitud a grandes rasgos:

:::image type="content" source="./media/request-units/request-units.png" alt-text="Unidades de solicitud consumidas por operaciones de base de datos" border="false":::

Para administrar y planear la capacidad, Azure Cosmos DB garantiza que el número de RU para una operación de base de datos determinada en un conjunto de datos concreto sea determinista. Puede examinar el encabezado de respuesta para realizar un seguimiento del número de RU consumidas por cualquier operación de base de datos. Una vez comprendidos los [factores que afectan a los cargos de unidad de solicitud](request-units.md#request-unit-considerations) y los requisitos de rendimiento de la aplicación, se puede ejecutar la aplicación de forma rentable.

El aprovisionamiento del número de RU para la aplicación se realiza por segundos en incrementos de 100 RU/segundo. Para escalar el rendimiento aprovisionado para la aplicación, puede aumentar o disminuir el número de RU en cualquier momento. Se puede escalar en incrementos o disminuciones de 100 RU. Puede realizar los cambios mediante programación o en Azure Portal. Se factura por hora.

Puede aprovisionar el rendimiento en dos granularidades diferentes:

* **Contenedores**: Para obtener más información, consulte [la manera de aprovisionar el rendimiento en un contenedor de Azure Cosmos](how-to-provision-container-throughput.md).
* **Bases de datos**: Para obtener más información, consulte [la manera de aprovisionar el rendimiento en una base de datos de Azure Cosmos](how-to-provision-database-throughput.md).

## <a name="request-unit-considerations"></a>Consideraciones de la unidad de solicitud

Al calcular el número de RU por segundo para aprovisionar, tenga en cuenta los siguientes factores:

* **Tamaño del elemento**: a medida que el tamaño de un elemento aumenta, el número de unidades de solicitud consumidas para leer o escribir el elemento también aumenta.

* **Indexación del elemento**: De forma predeterminada, cada elemento se indexa automáticamente. Se consumen menos unidades de solicitud si decide no indexar algunos elementos de un contenedor.

* **Recuento de propiedades del elemento**: suponiendo que se aplica la indexación predeterminada en todas las propiedades, el número de unidades de solicitud consumidas para escribir un elemento aumenta a medida que lo hace el recuento de propiedades del elemento.

* **Propiedades indexadas**: Una directiva de índice en cada contenedor determina qué propiedades se indexan de forma predeterminada. Para reducir el consumo de unidades de solicitud para operaciones de escritura, limite el número de las propiedades indexadas.

* **Coherencia de los datos**: los niveles de coherencia alta y de obsolescencia limitada consumen aproximadamente dos veces más unidades de solicitud al realizar operaciones de lectura en comparación con las que consumen los niveles de coherencia relajada.

* **Tipo de lecturas**: Las lecturas puntuales cuestan significativamente menos RU que las consultas.

* **Patrones de consultas**: la complejidad de una consulta afecta a la cantidad de unidades de solicitud consumidas en una operación. Entre los factores que influyen en el costo de las operaciones de consulta están los siguientes: 
    
    - El número de resultados de la consulta.
    - El número de predicados.
    - La naturaleza de los predicados.
    - El número de funciones definidas por el usuario.
    - El tamaño de los datos de origen.
    - El tamaño del conjunto de resultados.
    - Proyecciones

  Azure Cosmos DB garantiza que la misma consulta de los mismos datos cuesta siempre el mismo número de unidades de solicitud en ejecución repetidas.

* **Uso de scripts**: como las consultas, los procedimientos almacenados y los desencadenadores consumen unidades de solicitud según la complejidad de las operaciones que lleven a cabo. Cuando desarrolle la aplicación, inspeccione el [encabezado de cargo de solicitud](optimize-cost-queries.md#evaluate-request-unit-charge-for-a-query) para entender mejor cuánta capacidad de unidades de solicitud consume cada operación.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre el [aprovisionamiento del rendimiento de contenedores y bases de datos de Azure Cosmos](set-throughput.md).
* Más información sobre las [particiones lógicas](partition-data.md).
* Obtenga más información sobre el [escalado global del rendimiento aprovisionado](scaling-throughput.md).
* Obtenga más información sobre el [aprovisionamiento del rendimiento de un contenedor de Azure Cosmos](how-to-provision-container-throughput.md).
* Obtenga más información sobre el [aprovisionamiento del rendimiento de una base de datos de Azure Cosmos](how-to-provision-database-throughput.md).
* Obtenga información sobre cómo [encontrar el cargo de unidades de solicitud de una operación](find-request-unit-charge.md).
* Obtenga información sobre cómo [optimizar el costo de rendimiento aprovisionado en Azure Cosmos DB](optimize-cost-throughput.md).
* Obtenga información sobre cómo [optimizar el costo de lecturas y escrituras en Azure Cosmos DB](optimize-cost-reads-writes.md).
* Obtenga información sobre cómo [optimizar el costo de consultar en Azure Cosmos DB](optimize-cost-queries.md).
* Obtenga información sobre cómo [usar métricas para supervisar el rendimiento](use-metrics.md).
