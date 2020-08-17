---
title: Arquitectura de SQL de Synapse
description: Obtenga información acerca de cómo SQL de Azure Synapse combina el procesamiento paralelo masivo (MPP) con Azure Storage para lograr un alto rendimiento y escalabilidad.
services: synapse-analytics
author: mlee3gsd
manager: rothja
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: ''
ms.date: 04/15/2020
ms.author: martinle
ms.reviewer: igorstan
ms.openlocfilehash: 9f2f3eee12bb8741f6d079f6f081a08f4e2db9b5
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87046864"
---
# <a name="azure-synapse-sql-architecture"></a>Arquitectura de SQL de Azure Synapse 

En este artículo se describen los componentes de la arquitectura de SQL de Synapse.

[!INCLUDE [preview](../includes/note-preview.md)]

## <a name="synapse-sql-architecture-components"></a>Componentes de la arquitectura de SQL de Synapse

SQL de Synapse aprovecha una arquitectura de escalabilidad horizontal para distribuir el procesamiento de cálculo de datos entre varios nodos. Como el proceso está separado del almacenamiento, se puede escalar con independencia de los datos del sistema. 

En el caso de un grupo de SQL, la unidad de escalado es una abstracción de la eficacia de proceso que se conoce como [unidad de almacenamiento de datos](resource-consumption-models.md). 

En el caso de SQL a petición, al ser sin servidor, el escalado se realiza automáticamente para adaptarse a los requisitos de los recursos de consulta. A medida que la topología cambia con el tiempo al agregar o quitar nodos o conmutaciones por error, se adapta a los cambios y se asegura de que la consulta tenga suficientes recursos y finalice correctamente. Por ejemplo, en la imagen siguiente se muestra SQL a petición con cuatro nodos de proceso para ejecutar una consulta.

![Arquitectura de SQL de Synapse](./media//overview-architecture/sql-architecture.png)

SQL de Synapse usa una arquitectura basada en nodos. Las aplicaciones se conectan y emiten comandos de T-SQL a un nodo de control, que es el único punto de entrada de SQL de Synapse. 

El nodo de control del grupo de SQL utiliza el motor de MPP que optimiza las consultas para el procesamiento en paralelo y, después, pasa las operaciones a los nodos de ejecución para hacer su trabajo en paralelo. 

El nodo de control de SQL a petición utiliza el motor de procesamiento de consultas distribuidas (DQP) para optimizar y orquestar la ejecución distribuida de consultas de usuario mediante su división en consultas más pequeñas que se ejecutarán en nodos de ejecución. Cada consulta pequeña se denomina "tarea", que representa una unidad de ejecución distribuida. Lee los archivos del almacenamiento, combina los resultados de otras tareas, agrupa u ordena los datos recuperados de otras tareas. 

Los nodos de ejecución almacenan todos los datos del usuario en Azure Storage y ejecutan las consultas en paralelo. El servicio de movimiento de datos (DMS) es un servicio interno de nivel de sistema que mueve datos entre los nodos según sea necesario para ejecutar consultas en paralelo y devolver resultados precisos. 

Con un almacenamiento y proceso desacoplados, cuando se usa SQL de Synapse, es posible aprovechar un tamaño independiente de la capacidad de proceso, más allá de sus necesidades de almacenamiento. Para el escalado de SQL a petición esto se realiza automáticamente mientras que para el grupo de SQL es posible:

* Aumentar o reducir la capacidad de proceso en un grupo de SQL (almacenamiento de datos), sin mover los datos.
* Pausar la capacidad de proceso mientras se dejan los datos intactos, por lo que solo paga por el almacenamiento.
* Reanudar la capacidad de proceso durante las horas operativas.

## <a name="azure-storage"></a>Azure Storage

SQL de Synapse aprovecha Azure Storage para proteger los datos del usuario. Puesto que los datos se almacenan y administran en Azure Storage, el consumo de almacenamiento se cobra aparte. 

SQL a petición le permite consultar los archivos de su lago de datos en modo de solo lectura, mientras que el grupo de SQL también le permite ingerir datos. Cuando se ingieren datos en el grupo de SQL, los datos se particionan en **distribuciones** para optimizar el rendimiento del sistema. Puede elegir qué modelo de particionamiento quiere usar para distribuir los datos cuando define la tabla. Se admiten estos patrones de particionamiento:

* Hash
* Round Robin
* Replicar

## <a name="control-node"></a>Nodo de control

El nodo de control es el cerebro de la arquitectura. Es el front-end que interactúa con todas las aplicaciones y conexiones. 

En un grupo de SQL, el motor de MPP se ejecuta en el nodo de control para optimizar y coordinar las consultas en paralelo. Al enviar una consulta T-SQL a un grupo de SQL, el nodo de control la transforma en consultas que se ejecutan en cada distribución en paralelo.

En SQL a petición, el motor de DQP se ejecuta en el nodo de control para optimizar y coordinar la ejecución distribuida de consultas de usuario mediante su división en consultas más pequeñas que se ejecutarán en nodos de ejecución. También asigna conjuntos de archivos para que cada nodo los procese.

## <a name="compute-nodes"></a>Nodos de proceso

Los nodos de proceso proporcionan la eficacia de cálculo. 

En un grupo de SQL, las distribuciones se asignan a nodos de ejecución para su procesamiento. Al pagar por más recursos de proceso, el grupo reasigna las distribuciones a los nodos de ejecución disponibles. El número de nodos de ejecución va de 1 a 60, y viene determinado por el nivel de servicio del grupo de SQL. Cada nodo de cálculo tiene un identificador de nodo que está visible en las vistas del sistema. Para ver el identificador del nodo de ejecución, busque la columna node_id en las vistas del sistema cuyos nombres comiencen por sys.pdw_nodes. Para obtener una lista de las vistas del sistema, consulte la [MPP system views](/sql/relational-databases/system-catalog-views/sql-data-warehouse-and-parallel-data-warehouse-catalog-views?view=azure-sqldw-latest) (Vistas del sistema de MPP).

En SQL a petición, a cada nodo de ejecución se le asigna una tarea y un conjunto de archivos en los que se ejecutará la tarea. La tarea es una unidad de ejecución de consulta distribuida que, en realidad, es parte de la consulta enviada por el usuario. El escalado automático está en vigor para asegurarse de que se utilicen suficientes nodos de ejecución para ejecutar la consulta del usuario.

## <a name="data-movement-service"></a>Servicio de movimiento de datos

Servicio de movimiento de datos (DMS) es la tecnología de transporte de datos del grupo de SQL, que coordina el movimiento de los datos entre los nodos de ejecución. Algunas consultas requieren el movimiento de datos para asegurarse de que las consultas paralelas devuelven resultados precisos. Cuando un movimiento de datos es necesario, DMS asegura que los datos adecuados llegan a la ubicación adecuada.

> [!VIDEO https://www.youtube.com/embed/PlyQ8yOb8kc]

## <a name="distributions"></a>Distribuciones

Una distribución es la unidad básica de almacenamiento y procesamiento de consultas en paralelo que se ejecutan en datos distribuidos en el grupo de SQL. Cuando el grupo de SQL ejecuta una consulta, el trabajo se divide en 60 consultas más pequeñas que se ejecutan en paralelo. 

Cada una de estas 60 consultas más pequeñas se ejecuta en una de las distribuciones de datos. Cada nodo de ejecución administra una o más de las 60 distribuciones. Un grupo de SQL con recursos de proceso máximos tiene una distribución por nodo de proceso. Un grupo de SQL con recursos de proceso mínimos tiene todas las distribuciones en un nodo de proceso. 

## <a name="hash-distributed-tables"></a>Tablas distribuidas mediante una función hash
Una tabla con distribución por hash puede ofrecer el máximo rendimiento de consultas para combinaciones y agregaciones en tablas grandes. 

Para particionar los datos en una tabla con distribución por hash, un grupo de SQL emplea una función hash para asignar de una manera determinista cada fila a una distribución. En la definición de tabla, una de las columnas se designa como columna de distribución. La función hash usa el valor de la columna de distribución para asignar cada fila a una distribución.

El siguiente diagrama muestra cómo se almacena una tabla completa (no distribuida) como una tabla distribuida mediante una función hash. 

![Tabla distribuida](media//overview-architecture/hash-distributed-table.png "Tabla distribuida") 

* Cada fila pertenece a una distribución. 
* Un algoritmo hash determinista asigna cada fila a una distribución. 
* El número de filas de la tabla por cada distribución varía, lo que se hace patente en los diferentes tamaños de tablas.

Es preciso tener en cuenta consideraciones de rendimiento al seleccionar una columna de distribución, tales como la diferenciación, la asimetría de datos o los tipos de consultas que se ejecutan en el sistema.

## <a name="round-robin-distributed-tables"></a>Tablas distribuidas con el método round robin

Una tabla round robin es la tabla más sencilla de crear y ofrece un rendimiento rápido cuando se usa como tabla de almacenamiento provisional para las cargas.

Una tabla distribuida con el método round robin distribuye los datos uniformemente en la tabla, pero sin ninguna optimización adicional. Una distribución se elige primero de manera aleatoria y, después, los búferes de filas se asignan a las distribuciones secuencialmente. Es rápido cargar datos en una tabla round robin, pero el rendimiento de las consultas puede mejorar con tablas con distribución por hash. Las combinaciones de tablas round robin requieren reconstruir los datos, y esto requiere tiempo adicional.

## <a name="replicated-tables"></a>Tablas replicadas
Una tabla replicada proporciona el rendimiento de consultas más rápido para tablas pequeñas.

Una tabla que se replica tiene una copia completa de la tabla almacenada en la caché de cada nodo de proceso. Por lo tanto, al replicar una tabla se elimina la necesidad de transferir sus datos de un nodo de proceso a otro antes de una combinación o agregación. Las tablas replicadas se usan mejor con tablas pequeñas. Se requiere almacenamiento adicional y hay sobrecargas adicionales que se producen al escribir datos que hacen que las tablas grandes sean poco prácticas. 

En el diagrama siguiente se muestra una tabla replicada que se almacena en caché en la primera distribución de cada nodo de proceso. 

![Tabla replicada](media/overview-architecture/replicated-table.png "Tabla replicada") 

## <a name="next-steps"></a>Pasos siguientes

Ahora que ya conoce un poco sobre SQL de Synapse, aprenda a [crear un grupo de SQL](../quickstart-create-sql-pool-portal.md) rápidamente y a [cargar datos de ejemplo](../sql-data-warehouse/sql-data-warehouse-load-from-azure-blob-storage-with-polybase.md) (./sql-data-warehouse-load-sample-databases.md). O bien, inicie [mediante SQL a petición](../quickstart-sql-on-demand.md). Si no está familiarizado con Azure, el [Glosario de Azure](../../azure-glossary-cloud-terminology.md) le puede resultar útil para consultar la nueva terminología que se encuentre. 
