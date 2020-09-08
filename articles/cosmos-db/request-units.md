---
title: Unidades de solicitud como moneda de rendimiento en Azure Cosmos DB
description: Obtenga información sobre cómo especificar y estimar los requisitos de las unidades de solicitud en Azure Cosmos DB.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 08/19/2020
ms.openlocfilehash: 6831cb3f39c25eb69d16300156f456980cf57fa0
ms.sourcegitcommit: d661149f8db075800242bef070ea30f82448981e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88604841"
---
# <a name="request-units-in-azure-cosmos-db"></a>Unidades de solicitud en Azure Cosmos DB

Azure Cosmos DB admite varias API, como SQL, MongoDB, Cassandra, Gremlin y Table. Cada API tiene su propio conjunto de operaciones de base de datos. Estas abarcan desde sencillas lecturas y escrituras de punto hasta consultas complejas. Cada operación de base de datos consume recursos del sistema en función de la complejidad de la operación.

Azure Cosmos DB se encarga de normalizar el costo de todas las operaciones de base de datos y se expresa en términos de *unidades de solicitud* (RU en su forma abreviada). Puede considerar que las unidades de solicitud son como una moneda de rendimiento, que resume los recursos del sistema, como CPU, IOPS y memoria, necesarios para realizar las operaciones de base de datos compatibles con Azure Cosmos DB.

El costo de hacer una lectura puntual (es decir, capturar un solo elemento por su identificador y valor de clave de partición) para un elemento de 1 KB es 1 unidad de solicitud (o 1 RU). A todas las demás operaciones de base de datos se les asigna de forma similar un costo en términos de unidades de solicitud. Con independencia de qué API utilice para interactuar con el contenedor de Azure Cosmos, los costos siempre se miden por RU. Si la operación de base de datos es una escritura, lectura puntual o consulta, los costos siempre se miden en RU.

En la siguiente imagen se muestra el concepto de unidades de solicitud a grandes rasgos:

:::image type="content" source="./media/request-units/request-units.png" alt-text="Unidades de solicitud consumidas por operaciones de base de datos" border="false":::

Para administrar y planear la capacidad, Azure Cosmos DB garantiza que el número de RU para una operación de base de datos determinada en un conjunto de datos concreto sea determinista. Puede examinar el encabezado de respuesta para realizar un seguimiento del número de RU consumidas por cualquier operación de base de datos. Una vez comprendidos los [factores que afectan a los cargos de unidad de solicitud](request-units.md#request-unit-considerations) y los requisitos de rendimiento de la aplicación, se puede ejecutar la aplicación de forma rentable.

El tipo de cuenta de Azure Cosmos que está utilizando determina el modo en que se cobrarán los RU consumidos:

- En el modo [rendimiento aprovisionado](set-throughput.md), el aprovisionamiento del número de RU para la aplicación se realiza por segundos en incrementos de 100 RU/segundo. Para escalar el rendimiento aprovisionado para la aplicación, puede aumentar o disminuir el número de RU en cualquier momento, en incrementos o decrementos de 100 RU. Puede realizar los cambios mediante programación o en Azure Portal. Se le facturará cada hora por la cantidad de RU por segundo que haya aprovisionado. Puede aprovisionar el rendimiento en dos granularidades diferentes:
  - **Contenedores**: Para obtener más información, consulte [la manera de aprovisionar el rendimiento en un contenedor de Azure Cosmos](how-to-provision-container-throughput.md).
  - **Bases de datos**: Para obtener más información, consulte [la manera de aprovisionar el rendimiento en una base de datos de Azure Cosmos](how-to-provision-database-throughput.md).
- En el modo [sin servidor](serverless.md), no tiene que aprovisionar ningún rendimiento al crear recursos en la cuenta de Azure Cosmos. Al final del período de facturación, se le factura el número de unidades de solicitud consumidas por las operaciones de base de datos.

## <a name="request-unit-considerations"></a>Consideraciones de la unidad de solicitud

Al calcular el número de RU por segundo consumidos por su carga, tenga en cuenta los siguientes factores:

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
* Más información sobre [Azure Cosmos DB sin servidor](serverless.md).
* Más información sobre las [particiones lógicas](partition-data.md).
* Obtenga más información sobre el [escalado global del rendimiento aprovisionado](scaling-throughput.md).
* Obtenga más información sobre el [aprovisionamiento del rendimiento de un contenedor de Azure Cosmos](how-to-provision-container-throughput.md).
* Obtenga más información sobre el [aprovisionamiento del rendimiento de una base de datos de Azure Cosmos](how-to-provision-database-throughput.md).
* Obtenga información sobre cómo [encontrar el cargo de unidades de solicitud de una operación](find-request-unit-charge.md).
* Obtenga información sobre cómo [optimizar el costo de rendimiento aprovisionado en Azure Cosmos DB](optimize-cost-throughput.md).
* Obtenga información sobre cómo [optimizar el costo de lecturas y escrituras en Azure Cosmos DB](optimize-cost-reads-writes.md).
* Obtenga información sobre cómo [optimizar el costo de consultar en Azure Cosmos DB](optimize-cost-queries.md).
* Obtenga información sobre cómo [usar métricas para supervisar el rendimiento](use-metrics.md).
