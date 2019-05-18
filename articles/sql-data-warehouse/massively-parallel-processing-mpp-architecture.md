---
title: 'Azure SQL Data Warehouse: arquitectura de MPP | Microsoft Docs'
description: Obtenga información acerca de cómo Azure SQL Data Warehouse combina procesamiento paralelo masivo (MPP) con el almacenamiento de Azure para lograr un alto rendimiento y escalabilidad.
services: sql-data-warehouse
author: happynicolle
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.subservice: design
ms.date: 04/17/2018
ms.author: nicw
ms.reviewer: igorstan
ms.openlocfilehash: c3cdd464dffc810e76cf101ac70c2a14bbc4f9ff
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65790712"
---
# <a name="azure-sql-data-warehouse---massively-parallel-processing-mpp-architecture"></a>Azure SQL Data Warehouse: Arquitectura de procesamiento paralelo masivo (MPP)
Obtenga información acerca de cómo Azure SQL Data Warehouse combina procesamiento paralelo masivo (MPP) con el almacenamiento de Azure para lograr un alto rendimiento y escalabilidad. 

> [!VIDEO https://www.youtube.com/embed/PlyQ8yOb8kc]

## <a name="mpp-architecture-components"></a>Componentes de la arquitectura de MPP
SQL Data Warehouse aprovecha una arquitectura de escalado horizontal para distribuir el procesamiento de cálculo de datos en varios nodos. La unidad de escalado es una abstracción de la eficacia de proceso que se conoce como [unidad de almacenamiento de datos](what-is-a-data-warehouse-unit-dwu-cdwu.md). SQL Data Warehouse separa el proceso del almacenamiento, lo que permite escalar los procesos independientemente de los datos en el sistema.

![Arquitectura de SQL Data Warehouse](media/massively-parallel-processing-mpp-architecture/massively-parallel-processing-mpp-architecture.png)

SQL Data Warehouse usa una arquitectura basada en nodos. Las aplicaciones conectan y emiten comandos de T-SQL para un nodo de control, que es el único punto de entrada para el almacenamiento de datos. El nodo de control ejecuta el motor de MPP que optimiza las consultas para el procesamiento en paralelo y, después, pasa las operaciones a los nodos de ejecución para hacer su trabajo en paralelo. Los nodos de ejecución almacenan todos los datos del usuario en Azure Storage y ejecutan las consultas en paralelo. El servicio de movimiento de datos (DMS) es un servicio interno de nivel de sistema que mueve datos entre los nodos según sea necesario para ejecutar consultas en paralelo y devolver resultados precisos. 

Con el almacenamiento y el proceso separados, SQL Data Warehouse hace lo siguiente:

* Cambiar la potencia del proceso independientemente de las necesidades de almacenamiento.
* Aumentar o reducir la capacidad de proceso sin mover datos.
* Pausar la capacidad de proceso mientras se dejan los datos intactos, por lo que solo paga por el almacenamiento.
* Reanudar la capacidad de proceso durante las horas operativas.

### <a name="azure-storage"></a>Almacenamiento de Azure
SQL Data Warehouse usa Azure Storage para proteger los datos del usuario.  Puesto que los datos se almacenan y administran en Azure Storage, SQL Data Warehouse cobra por separado el consumo de almacenamiento. Los datos en sí mismos están particionados en **distribuciones** para optimizar el rendimiento del sistema. Puede elegir qué modelo de particionamiento quiere usar para distribuir los datos cuando define la tabla. SQL Data Warehouse admite estos patrones de particionamiento:

* Hash
* Round Robin
* Replicar

### <a name="control-node"></a>Nodo de control

El nodo de control es el cerebro del almacenamiento de datos. Es el front-end que interactúa con todas las aplicaciones y conexiones. El motor de MPP se ejecuta en el nodo de control para optimizar y coordinar las consultas en paralelo. Al enviar una consulta T-SQL a SQL Data Warehouse, el nodo de control la transforma en consultas que se ejecutarán en cada distribución en paralelo.

### <a name="compute-nodes"></a>Nodos de proceso

Los nodos de proceso proporcionan la eficacia de cálculo. Las distribuciones se asignan a nodos de proceso para su procesamiento. Al pagar más recursos de proceso, SQL Data Warehouse vuelve a asignar las distribuciones a los nodos de proceso disponibles. El número de nodos de proceso va de 1 a 60, y viene determinado por el nivel de servicio del almacenamiento de datos.

Cada nodo de cálculo tiene un identificador de nodo que está visible en las vistas del sistema. Para ver el identificador del nodo de ejecución, busque la columna node_id en las vistas del sistema cuyos nombres comiencen por sys.pdw_nodes. Para obtener una lista de las vistas del sistema, consulte la [MPP system views](https://docs.microsoft.com/sql/relational-databases/system-catalog-views/sql-data-warehouse-and-parallel-data-warehouse-catalog-views?view=aps-pdw-2016-au7) (Vistas del sistema de MPP).

### <a name="data-movement-service"></a>Servicio de movimiento de datos
Servicio de movimiento de datos (DMS) es la tecnología de transporte de datos que coordina el movimiento de los datos entre los nodos de proceso. Algunas consultas requieren el movimiento de datos para asegurarse de que las consultas paralelas devuelven resultados precisos. Cuando un movimiento de datos es necesario, DMS asegura que los datos adecuados llegan a la ubicación adecuada. 

## <a name="distributions"></a>Distribuciones

Una distribución es la unidad básica de almacenamiento y procesamiento de consultas en paralelo que se ejecutan en datos distribuidos. Cuando SQL Data Warehouse ejecuta una consulta, el trabajo se divide en 60 consultas más pequeñas que se ejecutan en paralelo. Cada una de estas 60 consultas más pequeñas se ejecuta en una de las distribuciones de datos. Cada nodo de ejecución administra una o más de las 60 distribuciones. Un almacén de datos con recursos de proceso máximos tiene una distribución por nodo de proceso. Un almacén de datos con recursos de proceso mínimos tiene todas las distribuciones en un nodo de proceso.  

## <a name="hash-distributed-tables"></a>Tablas distribuidas mediante una función hash
Una tabla con distribución por hash puede ofrecer el máximo rendimiento de consultas para combinaciones y agregaciones en tablas grandes. 

Para particionar datos en una tabla con distribución por hash, SQL Data Warehouse usa una función hash para asignar de una manera determinista cada fila a una distribución. En la definición de tabla, una de las columnas se designa como columna de distribución. La función hash usa el valor de la columna de distribución para asignar cada fila a una distribución.

El siguiente diagrama muestra cómo se almacena una tabla completa (no distribuida) como una tabla distribuida mediante una función hash. 

![Tabla distribuida](media/sql-data-warehouse-distributed-data/hash-distributed-table.png "Tabla distribuida")  

* Cada fila pertenece a una distribución.  
* Un algoritmo hash determinista asigna cada fila a una distribución.  
* El número de filas de la tabla por cada distribución varía, lo que se hace patente en los diferentes tamaños de tablas.

Es preciso tener en cuenta consideraciones de rendimiento al seleccionar una columna de distribución, tales como la diferenciación, la asimetría de datos o los tipos de consultas que se ejecutan en el sistema.

## <a name="round-robin-distributed-tables"></a>Tablas distribuidas con el método round robin
Una tabla round robin es la tabla más sencilla de crear y ofrece un rendimiento rápido cuando se usa como tabla de almacenamiento provisional para las cargas.

Una tabla distribuida con el método round robin distribuye los datos uniformemente en la tabla, pero sin ninguna optimización adicional. Una distribución se elige primero de manera aleatoria y, después, los búferes de filas se asignan a las distribuciones secuencialmente. Es rápido cargar datos en una tabla round robin, pero el rendimiento de las consultas puede mejorar con tablas con distribución por hash. Las combinaciones de tablas round robin suelen requerir reconstruir los datos y esto requiere tiempo adicional.


## <a name="replicated-tables"></a>Tablas replicadas
Una tabla replicada proporciona el rendimiento de consultas más rápido para tablas pequeñas.

Una tabla que se replica tiene una copia completa de la tabla almacenada en la caché de cada nodo de proceso. Por lo tanto, al replicar una tabla se elimina la necesidad de transferir sus datos de un nodo de proceso a otro antes de una combinación o agregación. Las tablas replicadas se usan mejor con tablas pequeñas. Se requiere almacenamiento adicional y hay una sobrecarga adicional que se incurre al escribir datos que hacen que las tablas grandes que no sea práctico.  

En el diagrama siguiente se muestra una tabla replicada. Para SQL Data Warehouse, se guarda una copia de la tabla replicada en la caché en la primera distribución en cada nodo de ejecución.  

![Tabla replicada](media/sql-data-warehouse-distributed-data/replicated-table.png "Tabla replicada") 

## <a name="next-steps"></a>Pasos siguientes
Ahora que ya conoce un poco SQL Data Warehouse, aprenda a [crear una instancia de SQL Data Warehouse][create a SQL Data Warehouse] y [cargar datos de ejemplo de manera rápida][load sample data]. Si no está familiarizado con Azure, el [Glosario de Azure][Azure glossary] le puede resultar útil para consultar la nueva terminología que se encuentre. O bien, examine algunos de estos otros recursos de SQL Data Warehouse.  

* [Casos de éxito de clientes]
* [Blogs]
* [Solicitud de función]
* [Vídeos]
* [Blogs de Customer Advisory Team]
* [Creación de una incidencia de soporte técnico]
* [Foro de MSDN]
* [Foro Stack Overflow]
* [Twitter]

<!--Image references-->
[1]: ./media/sql-data-warehouse-overview-what-is/dwarchitecture.png

<!--Article references-->
[Creación de una incidencia de soporte técnico]: ./sql-data-warehouse-get-started-create-support-ticket.md
[load sample data]: ./sql-data-warehouse-load-sample-databases.md
[create a SQL Data Warehouse]: ./sql-data-warehouse-get-started-provision.md
[Migration documentation]: ./sql-data-warehouse-overview-migrate.md
[SQL Data Warehouse solution partners]: ./sql-data-warehouse-partner-business-intelligence.md
[Integrated tools overview]: ./sql-data-warehouse-overview-integrate.md
[Backup and restore overview]: ./sql-data-warehouse-restore-database-overview.md
[Azure glossary]: ../azure-glossary-cloud-terminology.md

<!--MSDN references-->

<!--Other Web references-->
[Casos de éxito de clientes]: https://azure.microsoft.com/case-studies/?service=sql-data-warehouse
[Blogs]: https://azure.microsoft.com/blog/tag/azure-sql-data-warehouse/
[Blogs de Customer Advisory Team]: https://blogs.msdn.microsoft.com/sqlcat/tag/sql-dw/
[Solicitud de función]: https://feedback.azure.com/forums/307516-sql-data-warehouse
[Foro de MSDN]: https://social.msdn.microsoft.com/Forums/azure/home?forum=AzureSQLDataWarehouse
[Foro Stack Overflow]: https://stackoverflow.com/questions/tagged/azure-sqldw
[Twitter]: https://twitter.com/hashtag/SQLDW
[Vídeos]: https://azure.microsoft.com/documentation/videos/index/?services=sql-data-warehouse
[SLA for SQL Data Warehouse]: https://azure.microsoft.com/support/legal/sla/sql-data-warehouse/v1_0/
[Volume Licensing]: https://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=37
[Service Level Agreements]: https://azure.microsoft.com/support/legal/sla/
