---
title: Compatibilidad de Gremlin de Azure Cosmos DB con las características de TinkerPop
description: Documentación de referencia para problemas de compatibilidad del motor Graph
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: reference
ms.date: 09/10/2019
ms.author: sngun
ms.openlocfilehash: 7257246f618e3028534f3ebd60eaf6f94a3a4720
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87092514"
---
# <a name="azure-cosmos-db-gremlin-compatibility"></a>Compatibilidad de Gremlin de Azure Cosmos DB
El motor de Azure Cosmos DB Graph sigue estrechamente la especificación de los pasos de recorrido de [Apache TinkerPop](https://tinkerpop.apache.org/docs/current/reference/#graph-traversal-steps), pero hay diferencias en la implementación específicas de Azure Cosmos DB. Para obtener la lista de los pasos de Gremlin admitidos, consulte el artículo [Compatibilidad con el protocolo de transferencia de la API Gremlin](gremlin-support.md).

## <a name="behavior-differences"></a>Diferencias de comportamiento

* El motor Graph de Azure Cosmos DB ejecuta el recorrido centrando la ***prioridad en la amplitud***, mientras que Gremlin de TinkerPop lo hace centrando la prioridad en la profundidad. Este comportamiento logra un mejor rendimiento en un sistema escalable horizontalmente como Cosmos DB. 

## <a name="unsupported-features"></a>Características no admitidas

* ***[Gremlin Bytecode](https://tinkerpop.apache.org/docs/current/tutorials/gremlin-language-variants/)*** es una especificación independiente del lenguaje de programación para recorridos de grafos. Cosmos DB Graph todavía no lo admite. Use `GremlinClient.SubmitAsync()` y pase el recorrido como una cadena de texto.

* Actualmente, no se admite el establecimiento de la cardinalidad ***`property(set, 'xyz', 1)`***. En su lugar, use `property(list, 'xyz', 1)`. Para más información, consulte [Propiedades Vertex con TinkerPop](http://tinkerpop.apache.org/docs/current/reference/#vertex-properties).

* El ***paso `match()`*** no se encuentra disponible actualmente. Este paso proporciona funcionalidades de consulta declarativa.

* No se admiten los ***objetos como propiedades*** en bordes ni vértices. Las propiedades solo pueden ser tipos primitivos o matrices.

* No se admite la ***ordenación por propiedades de la matriz*** `order().by(<array property>)`. Solo se admite la ordenación por tipos primitivos.

* No se admiten los ***tipos JSON no primitivos***. Use los tipos `string`, `number` o `true`/`false`. No se admiten los valores `null`. 

* El serializador ***GraphSONv3*** no se admite actualmente. Use las clases Serializer, Reader y Writer de `GraphSONv2` en la configuración de conexión. Los resultados devueltos por la API de Azure Cosmos DB Gremlin no tienen el mismo formato que el formato GraphSON. 

* Actualmente no se admiten las **expresiones y funciones lambda**. Esto incluye las funciones `.map{<expression>}`, `.by{<expression>}` y `.filter{<expression>}`. Para obtener más información y saber cómo volver a escribirlas con los pasos de Gremlin, vea [Una nota sobre las expresiones lambda](http://tinkerpop.apache.org/docs/current/reference/#a-note-on-lambdas).

* No se admiten las ***transacciones*** debido a la naturaleza distribuida del sistema.  Configure el modelo de coherencia adecuado en la cuenta de Gremlin para "leer sus propias escrituras", y use la simultaneidad optimista para resolver las escrituras conflictivas.

## <a name="known-limitations"></a>Restricciones conocidas

* **Uso de índices para las consultas de Gremlin con pasos `.V()` de recorrido intermedio**: actualmente, solo la primera llamada a `.V()` de un recorrido usará el índice para resolver los filtros o predicados asociados a él. Las llamadas subsiguientes no consultarán el índice, lo que podría aumentar la latencia y el costo de la consulta.
    
    Suponiendo que la indexación predeterminada, una consulta de Gremlin de lectura típica que se inicia con el paso `.V()` usaría parámetros en sus pasos de filtrado asociados, como `.has()` o `.where()`, para optimizar el costo y el rendimiento de la consulta. Por ejemplo:

    ```java
    g.V().has('category', 'A')
    ```

    Sin embargo, cuando se incluye más de un paso `.V()` en la consulta de Gremlin, la resolución de los datos para la consulta podría no ser la óptima. Tome la siguiente consulta como ejemplo:

    ```java
    g.V().has('category', 'A').as('a').V().has('category', 'B').as('b').select('a', 'b')
    ```

    Esta consulta devolverá dos grupos de vértices basados en su propiedad llamada `category`. En este caso, solo la primera llamada, `g.V().has('category', 'A')`, usará el índice para resolver los vértices en función de los valores de sus propiedades.

    Una solución alternativa para esta consulta es usar pasos subtransversales como `.map()` y `union()`. Esto se muestra a continuación:

    ```java
    // Query workaround using .map()
    g.V().has('category', 'A').as('a').map(__.V().has('category', 'B')).as('b').select('a','b')

    // Query workaround using .union()
    g.V().has('category', 'A').fold().union(unfold(), __.V().has('category', 'B'))
    ```

    Puede revisar el rendimiento de las consultas mediante el [paso de Gremlin `executionProfile()`](graph-execution-profile.md).

## <a name="next-steps"></a>Pasos siguientes
* Visite la página [Voz del usuario de Cosmos DB](https://feedback.azure.com/forums/263030-azure-cosmos-db) para compartir comentarios y ayudar al equipo a centrarse en las características que son importantes para usted.
