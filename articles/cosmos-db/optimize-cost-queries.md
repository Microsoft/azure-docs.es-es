---
title: Optimización del costo y las RU/s para ejecutar consultas en Azure Cosmos DB
description: Aprenda a evaluar los cargos por las unidades de solicitud y a optimizar la consulta en términos de rendimiento y costo.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 07/24/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: c6ff67e3c1588f6671173d6fa1eec3dc15883291
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89020922"
---
# <a name="optimize-query-cost-in-azure-cosmos-db"></a>Optimización de los costos de consulta de Azure Cosmos DB

Azure Cosmos DB ofrece un amplio conjunto de operaciones de base de datos, incluidas las consultas jerárquicas y relacionales que se realizan en los elementos de un contenedor. El costo asociado a cada una de estas operaciones variará en función de la CPU, la E/S y la memoria necesarias para completar la operación. En lugar de administrar y pensar sobre los recursos de hardware, puede pensar en una unidad de solicitud (RU) como una medida única para los recursos necesarios para realizar varias operaciones de la base de datos y dar servicio a una solicitud. En este artículo se describe la evaluación de los cargos por las unidades de solicitud y la optimización de la consulta en términos de rendimiento y costo.

Las lecturas en Azure Cosmos DB se suelen ordenar de la más rápida o eficiente a la más lenta o menos eficiente en cuanto a capacidad de proceso, de la siguiente manera:  

* Lecturas puntuales (búsqueda de clave y valor en un único id. de elemento y clave de partición).

* Consulta con una cláusula de filtro en una única clave de partición.

* Consulta sin una cláusula de filtro de igualdad o intervalo en cualquier propiedad.

* Consulta sin filtros.

Dado que las búsquedas de clave y valor en el id. de elemento son el tipo de lectura más eficiente, debe asegurarse de que el identificador de elemento tiene un valor significativo.

Las consultas que leen datos de una o varias particiones incurren en una latencia mayor y consumen un número mayor de unidades de solicitud. Puesto que cada partición tiene indexación automática en todas las propiedades, la consulta se puede atender eficazmente desde el índice. Puede realizar las consultas que abarcan varias particiones más rápido mediante las opciones de paralelismo. Para aprender más sobre la creación de particiones y las claves de particiones, consulte [Creación de particiones en Azure Cosmos DB](partitioning-overview.md).

## <a name="evaluate-request-unit-charge-for-a-query"></a>Evaluación del cargo de unidades de solicitud para una consulta

Una vez que haya almacenado algunos datos en los contenedores de Azure Cosmos, puede usar Data Explorer de Azure Portal para construir y ejecutar las consultas. Con el explorador de datos también puede obtener el costo de las consultas. Este método le dará una idea de los cargos reales relacionados con las consultas típicas y las operaciones que admite el sistema.

También puede obtener el costo de las consultas mediante programación usando los SDK. Para medir la sobrecarga de cualquier operación, como la creación, la actualización o la eliminación, consulte el encabezado `x-ms-request-charge` al usar la API REST. Si usa el SDK para .NET o para Java, la propiedad `RequestCharge` es equivalente a obtener el cargo de la solicitud y está presente en ResourceResponse o FeedResponse.

```csharp
// Measure the performance (request units) of writes
ResourceResponse<Document> response = await client.CreateDocumentAsync(collectionSelfLink, myDocument); 

Console.WriteLine("Insert of an item consumed {0} request units", response.RequestCharge); 

// Measure the performance (request units) of queries 
IDocumentQuery<dynamic> queryable = client.CreateDocumentQuery(collectionSelfLink, queryString).AsDocumentQuery(); 

while (queryable.HasMoreResults) 
     { 
          FeedResponse<dynamic> queryResponse = await queryable.ExecuteNextAsync<dynamic>(); 
          Console.WriteLine("Query batch consumed {0} request units", queryResponse.RequestCharge); 
     }
```

## <a name="factors-influencing-request-unit-charge-for-a-query"></a>Factores que influyen en el cargo de la unidad de solicitud para una consulta

Las unidades de solicitud para las consultas dependen de distintos factores. Por ejemplo, detalles sobre el número de elementos de Azure Cosmos cargado y devueltos, el número de búsquedas en el índice, el tiempo de compilación de consulta, etcétera. Azure Cosmos DB garantiza que la misma consulta de los mismos datos consumirá siempre el mismo número de unidades de solicitud, aunque se repitan las ejecuciones. El perfil de la consulta con métricas de ejecución de consultas ofrece una buena idea de cómo se invierten las unidades de solicitud.  

En algunos casos podría ver una secuencia de 200 y 429 respuestas y unidades de solicitud variables en la ejecución paginada de las consultas; esto se debe a que estas se ejecutan lo más rápido posible en función de las RU disponibles. Quizá vea una ejecución de consulta dividida en varias páginas o en recorrido de ida y vuelta entre el cliente y servidor. Por ejemplo, 10 000 elementos pueden devolverse como varias páginas y que cada una se cargue en función del cálculo realizado para esa página. Al sumar estas páginas debe obtener el mismo número de unidades de solicitud que obtendría para la consulta completa.  

## <a name="metrics-for-troubleshooting"></a>Métricas de solución de problemas

El rendimiento y la capacidad de proceso consumida por las consultas y las funciones definidas por el usuario (UDF) dependen principalmente del cuerpo de la función. La manera más sencilla de averiguar cuánto tiempo se dedica a la ejecución de consultas en la función definida por el usuario y el número de unidades de solicitud consumidas, es habilitar las métricas de consulta. Si usa el SDK de .NET, estas son las métricas de consulta de ejemplo que este devuelve:

```bash
Retrieved Document Count                 :               1              
Retrieved Document Size                  :           9,963 bytes        
Output Document Count                    :               1              
Output Document Size                     :          10,012 bytes        
Index Utilization                        :          100.00 %            
Total Query Execution Time               :            0.48 milliseconds 
  Query Preparation Times 
    Query Compilation Time               :            0.07 milliseconds 
    Logical Plan Build Time              :            0.03 milliseconds 
    Physical Plan Build Time             :            0.05 milliseconds 
    Query Optimization Time              :            0.00 milliseconds 
  Index Lookup Time                      :            0.06 milliseconds 
  Document Load Time                     :            0.03 milliseconds 
  Runtime Execution Times 
    Query Engine Execution Time          :            0.03 milliseconds 
    System Function Execution Time       :            0.00 milliseconds 
    User-defined Function Execution Time :            0.00 milliseconds 
  Document Write Time                    :            0.00 milliseconds 
  Client Side Metrics 
    Retry Count                          :               1              
    Request Charge                       :            3.19 RUs  
```

## <a name="best-practices-to-cost-optimize-queries"></a>Procedimientos recomendados para optimizar el costo de las consultas 

Al optimizar el costo de las consultas, considere los siguientes procedimientos recomendados:

* **Colocación de varios tipos de entidad**

   Intente colocar varios tipos de entidad dentro de un único contenedor o en el menor número de ellos posible. Este método produce beneficios no solo en lo relativo a los precios, sino también respecto a las transacciones y la ejecución de las consultas. Las consultas se limitan a un único contenedor y las transacciones atómicas con varios registros a través de procedimientos almacenados o desencadenadores, a una clave de partición de un único contenedor. Colocar las entidades en el mismo contenedor puede reducir el número de recorridos de ida y vuelta en la red para resolver las relaciones entre registros. Por lo tanto, aumenta el rendimiento de un extremo a otro, permite las transacciones atómicas con varios registros para un mayor conjunto de datos y, como resultado, reduce los costos. Si la colocación de varios tipos de entidad en un único contenedor o en un número reducido de estos resulta difícil en su caso, normalmente porque va a migrar una aplicación existente y no quiere realizar cambios de código, debe considerar el rendimiento del aprovisionamiento a nivel de la base de datos.  

* **Medición y optimización del uso menor de unidades de solicitud por segundo**

   La complejidad de una consulta afecta a la cantidad de unidades de solicitud (RU) consumidas para una operación. El número de predicados, la naturaleza de estos, el número de UDF y el tamaño del conjunto de datos de origen son factores que influyen en el costo de las operaciones de consulta. 

   El cargo de la solicitud devuelto en el encabezado de solicitud indica el costo de una consulta determinada. Por ejemplo, si una consulta devuelve 1000 elementos de 1 KB, el costo de la operación es 1000. Por lo tanto, al cabo de un segundo, el servidor atenderá solo dos de estas solicitudes antes de limitar la velocidad de las solicitudes posteriores. Para más información, consulte el artículo sobre las [unidades de solicitud](request-units.md) y la calculadora de unidades de solicitud. 

## <a name="next-steps"></a>Pasos siguientes

A continuación, puede seguir obteniendo más información sobre la optimización de costos en Azure Cosmos DB con los siguientes artículos:

* Sobre los [precios de Azure Cosmos DB](how-pricing-works.md)
* Obtenga más información sobre la [optimización para desarrollo y pruebas](optimize-dev-test.md).
* Obtenga más información sobre [la factura de Azure Cosmos DB](understand-your-bill.md).
* Obtenga más información sobre la [optimización del costo de la capacidad del rendimiento](optimize-cost-throughput.md).
* Obtenga más información sobre la [optimización del costo del almacenamiento](optimize-cost-storage.md).
* Obtenga más información sobre la [optimización del costo de la lectura y la escritura](optimize-cost-reads-writes.md).
* Obtenga más información sobre la [optimización del costo de las cuentas de Azure Cosmos de varias regiones](optimize-cost-regions.md).
* Sobre la [capacidad reservada de Azure Cosmos DB](cosmos-db-reserved-capacity.md)

