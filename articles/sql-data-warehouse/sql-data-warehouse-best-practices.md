---
title: Procedimientos recomendados para Azure SQL Data Warehouse | Microsoft Docs
description: Recomendaciones y procedimientos recomendados que debe saber para desarrollar soluciones de Azure SQL Data Warehouse.
services: sql-data-warehouse
author: happynicolle
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.subservice: design
ms.date: 11/26/2018
ms.author: nicw
ms.reviewer: igorstan
ms.openlocfilehash: 1e7fe0f7630e59be231f2513cdb7e38f8c15a68f
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65762653"
---
# <a name="best-practices-for-azure-sql-data-warehouse"></a>Procedimientos recomendados para Azure SQL Data Warehouse
Este artículo es una colección de procedimientos recomendados que le ayudará a conseguir un rendimiento óptimo de la instancia de Azure SQL Data Warehouse.  Algunos de los conceptos son básicos y fáciles de explicar, otros son más avanzados y solo se pueden ver por encima en este artículo.  El objetivo de este artículo es proporcionarle algunos consejos básicos y mostrarle los aspectos importantes que debe considerar al crear almacenamiento de datos.  En cada sección se presenta un concepto y se le indican artículos que lo desarrollan más en detalle.

Si acaba de empezar con Azure SQL Data Warehouse, no se preocupe por la amplitud del contenido de este artículo.  Los temas se van desarrollando básicamente en orden de importancia.  Un buen comienzo es centrarse en los primeros conceptos.  Según se vaya familiarizando y se sienta cómodo con SQL Date Warehouse, vuelva y eche un vistazo a los demás temas.  No tardará mucho en darle sentido a todo.

Para obtener una guía de carga, consulte la [guía para cargar datos](guidance-for-loading-data.md).

## <a name="reduce-cost-with-pause-and-scale"></a>Menos costos gracias a las características de pausa y escalado
Para más información acerca de cómo reducir los costos mediante la pausa y el escalado, consulte [Administración de procesos](sql-data-warehouse-manage-compute-overview.md). 


## <a name="maintain-statistics"></a>Mantenimiento de estadísticas
A diferencia de SQL Server, que automáticamente detecta y crea o actualiza las estadísticas en columnas, SQL Data Warehouse requiere el mantenimiento manual de las estadísticas.  Aunque nuestra intención es cambiar esto, por ahora deberá realizar el mantenimiento de las estadísticas para garantizar que los planes de SQL Data Warehouse son óptimos.  Los planes que crea el optimizador son igual de buenos que las estadísticas disponibles.  **Crear estadísticas de muestra en cada columna es una forma sencilla de empezar a trabajar con las estadísticas.**  Es igualmente importante actualizar las estadísticas cuando se produzcan cambios significativos en los datos.  Un enfoque conservador puede ser actualizar las estadísticas diariamente o después de cada carga.  Existen inconvenientes entre el rendimiento y el costo de crear y actualizar las estadísticas. Si cree que tarda demasiado en realizar el mantenimiento de todas las estadísticas, puede intentar ser más selectivo acerca de las columnas con estadísticas o las que necesitan actualizarse con frecuencia.  Por ejemplo, puede actualizar las columnas de fecha, donde se añadan valores todos los días. **Sacará el máximo provecho con las estadísticas en columnas relacionadas con combinaciones, columnas que se usan en la cláusula WHERE y columnas de GROUP BY.**

Consulte también [Administración de estadísticas en tablas en SQL Data Warehouse][Manage table statistics], [CREATE STATISTICS][CREATE STATISTICS] y [UPDATE STATISTICS][UPDATE STATISTICS]

## <a name="group-insert-statements-into-batches"></a>Agrupación de instrucciones INSERT en lotes
Una tabla pequeña se puede cargar perfectamente una sola vez con una instrucción INSERT o incluso con una recarga periódica que le funcione bien con una instrucción del tipo `INSERT INTO MyLookup VALUES (1, 'Type 1')`.  Sin embargo, si necesita cargar miles o millones de filas a lo largo del día, es posible que las instrucciones INSERT sencillas no sean suficientes.  En su lugar, desarrolle sus procesos para que se escriban en un archivo y otro proceso que se ejecute periódicamente y lo cargue.

Consulte también [INSERT][INSERT]

## <a name="use-polybase-to-load-and-export-data-quickly"></a>Uso de PolyBase para cargar y exportar los datos rápidamente
SQL Data Warehouse admite la carga y exportación de datos con varias herramientas, como Azure Data Factory, PolyBase y BCP.  Para pequeñas cantidades de datos donde el rendimiento no es clave, cualquier herramienta le sirve.  Sin embargo, para cargar o exportar grandes volúmenes de datos o si se necesita un rendimiento rápido, PolyBase es la mejor opción.  PolyBase está diseñado para aprovechar la estructura MPP (procesamiento masivo en paralelo) de SQL Data Warehouse y, por tanto, carga y exporta grandes cantidades de datos más rápido que cualquier otra herramienta.  Lo que haya cargado con PolyBase se ejecuta con la consulta CTAS o de selección.  **CTAS reduce el registro de transacciones y es la manera más rápida de cargar datos.**  Azure Data Factory también admite cargas de PolyBase y puede lograr un rendimiento similar a CTAS.  PolyBase admite distintos de formatos de archivo, como Gzip.  **Con el fin de conseguir un mayor rendimiento al usar archivos de texto gzip, divídalos en 60 o más archivos para aumentar el paralelismo de la carga.**  Para conseguir un rendimiento total más rápido, cargue los datos simultáneamente.

Consulte también [Carga de datos en Azure SQL Data Warehouse][Load data], [Guía para el uso de PolyBase en SQL Data Warehouse][Guide for using PolyBase], [Azure SQL Data Warehouse loading patterns and strategies][Azure SQL Data Warehouse loading patterns and strategies] (Patrones y estrategias de carga de Azure SQL Data Warehouse), [Carga de datos en SQL Data Warehouse con Data Factory][Load Data with Azure Data Factory], [Movimiento de datos hacia y desde SQL Data Warehouse mediante Azure Data Factory][Move data with Azure Data Factory], [CREATE EXTERNAL FILE FORMAT][CREATE EXTERNAL FILE FORMAT], [Create Table As Select (CTAS) en SQL Data Warehouse][Create table as select (CTAS)]

## <a name="load-then-query-external-tables"></a>Carga y consulta de tablas externas
Aunque Polybase, también conocido ahora como tablas externas, puede ser la manera más rápida de cargar datos, no resulta óptimo para las consultas. Las tablas de Polybase en SQL Data Warehouse solo admiten actualmente archivos de blobs de Azure y almacenamiento de Azure Data Lake Store. Estos archivos no tienen recursos de proceso que los respalde.  Como resultado, SQL Data Warehouse no puede descargar este trabajo y, por tanto, para leer los datos del archivo entero, debe cargarlo en tempdb.  Por lo tanto, si tiene varias consultas que van a consultar estos datos, es mejor cargarlos una vez que las consultas usen la tabla local.

Consulte también [Guía para el uso de PolyBase en SQL Data Warehouse][Guide for using PolyBase]

## <a name="hash-distribute-large-tables"></a>Distribución Hash para tablas grandes
De forma predeterminada, las tablas se distribuyen según el patrón Round Robin.  Esto facilita a los usuarios empezar a crear tablas sin tener que decidir sobre la distribución.  Las tablas round robin pueden ser suficientes para algunas cargas de trabajo, pero en la mayoría de los casos, la selección de una columna de distribución funcionará mucho opción.  El ejemplo más común de tabla distribuida por una columna que supera con creces a una Round Robin es al combinarse dos tablas grandes de hechos.  Por ejemplo, si tiene una tabla de pedidos, que se distribuye por order_id, y una tabla de transacciones, que también se distribuye por order_id, al unir la tabla de pedidos a la de transacciones en order_id, esta consulta se convierte en una consulta de paso a través, lo que significa que se eliminan las operaciones de movimiento de datos.  Menos pasos suponen consultas más rápidas.  Menos movimiento de datos también se traduce en consultas más rápidas.  Esta explicación es muy general. Al cargar una tabla con distribución, asegúrese de que no se ordenan los datos entrantes en la clave de distribución, ya que esto ralentizará la carga.  Consulte los siguientes vínculos para mucho más detalles sobre cómo seleccionar una columna de distribución puede mejorar el rendimiento, además de cómo definir una tabla distribuida en la cláusula WITH de la instrucción CREATE TABLE.

Consulte también [Información general de tablas en SQL Data Warehouse][Table overview], [Distribución de tablas en SQL Data Warehouse][Table distribution], [Selección de la distribución de tablas][Selecting table distribution], [CREATE TABLE][CREATE TABLE], [CREATE TABLE AS SELECT][CREATE TABLE AS SELECT]

## <a name="do-not-over-partition"></a>Sin particiones excesivas
Crear particiones de datos puede ser muy eficaz para el mantenimiento de los datos mediante la modificación de particiones o exámenes de optimización, pero el exceso de particiones puede ralentizar las consultas.  A menudo una estrategia de división con granularidad alta que puede funcionar bien en SQL Server no funciona correctamente en SQL Data Warehouse.  El exceso de particiones también puede reducir la eficacia de los índices de almacén de columnas agrupadas si cada partición tiene menos de 1 millón de filas.  Tenga en cuenta que, en segundo plano, SQL Data Warehouse divide los datos automáticamente en 60 bases de datos, por lo que si crea una tabla con 100 particiones, se generan realmente 6000 particiones.  Cada carga de trabajo es diferente, por lo mejor es probar con las particiones para ver qué funciona mejor para la suya.  Considere la posibilidad de reducir la granularidad respecto a lo que le funcionaba en SQL Server.  Por ejemplo, puede usar particiones semanales o mensuales, en lugar de diarias.

Consulte también cómo [Creación de particiones de tablas en SQL Data Warehouse][Table partitioning]

## <a name="minimize-transaction-sizes"></a>Reducción del tamaño de las transacciones
Las instrucciones INSERT, UPDATE Y DELETE se ejecutan en las transacciones y, cuando fallan, deben deshacerse.  Para que no se tarde tanto en deshacer, reduzca el tamaño de las transacciones siempre que pueda.  Puede hacerlo si divide las instrucciones INSERT, UPDATE y DELETE en partes.  Por ejemplo, si tiene una instrucción INSERT que se suele tardar de 1 hora, si puede, divídala en 4 partes de 15 minutos cada una.  Aproveche los casos de registro mínimo, como CTAS, TRUNCATE, DROP TABLE o INSERT para vaciar las tablas y así reducir el riesgo de reversión.  Otra manera de eliminar reversiones es usar funciones de solo metadatos, como la modificación de particiones para la administración de datos.  Por ejemplo, en lugar de ejecutar una instrucción DELETE para eliminar todas las filas de una tabla cuyo order_date fuera octubre de 2001, podría dividir los datos mensualmente y desactivar la división con los datos de una partición vacía de otra tabla (consulte los ejemplos de ALTER TABLE).  Para tablas sin divisiones, puede usar CTAS en lugar de DELETE para escribir los datos que quiera mantener en una tabla.  Si CTAS tarda lo mismo, es una operación mucho más segura, ya que su registro de transacciones es mínimo y se puede cancelar rápidamente si es necesario.

Consulte también [Transacciones en SQL Data Warehouse][Understanding transactions], [Optimización de transacciones para SQL Data Warehouse][Optimizing transactions], [Creación de particiones de tablas en SQL Data Warehouse][Table partitioning], [TRUNCATE TABLE][TRUNCATE TABLE], [ALTER TABLE][ALTER TABLE], [Create table as select (CTAS) en SQL Data Warehouse][Create table as select (CTAS)]

## <a name="use-the-smallest-possible-column-size"></a>Uso del tamaño de columna mínimo
Al definir el DDL, usar el tipo de datos mínimo compatible con los datos mejorará el rendimiento de la consulta.  Esto tiene especial importancia para las columnas CHAR y VARCHAR.  Si el valor mayor máximo de una columna es 25 caracteres, defina la columna como VARCHAR(25).  Evite definir todas las columnas de caracteres con una longitud predeterminada de gran tamaño.  Defina las columnas como VARCHAR en lugar de NVARCHAR cuando no se necesite nada más.

Consulte también [Información general de tablas en SQL Data Warehouse][Table overview], [Tipos de datos de las tablas en SQL Data Warehouse][Table data types], [CREATE TABLE][CREATE TABLE]

## <a name="use-temporary-heap-tables-for-transient-data"></a>Uso de tablas de apilamiento temporal para datos transitorios
Cuando almacene datos temporalmente en SQL Data Warehouse, las tablas de apilamiento pueden agilizar el proceso global.  Si solo carga datos para transformarlos después, cargar la tabla de apilamiento será mucho más rápido que cargar los datos en una tabla de almacén de columnas agrupadas.  Además, los datos de una tabla temporal también se cargarán mucho más rápido que las tablas de almacenamiento permanente.  Las tablas temporales empiezan por "#" y solo se puede acceder a ellas desde la sesión en la que se crean, por lo que pueden no funcionar en algunas situaciones.   Las tablas de apilamiento se definen en la cláusula WITH de CREATE TABLE.  Si usa una tabla temporal, no olvide crear estadísticas en ella también.

Consulte también [Tablas temporales en SQL Data Warehouse][Temporary tables], [CREATE TABLE][CREATE TABLE], [CREATE TABLE AS SELECT][CREATE TABLE AS SELECT]

## <a name="optimize-clustered-columnstore-tables"></a>Optimización de tablas de almacén de columnas agrupadas
Los índices de almacén de columnas agrupadas son una de las maneras más eficaces para almacenar datos en SQL Data Warehouse.  De forma predeterminada, las tablas de SQL Data Warehouse se crean como almacén de columnas agrupadas.  Para conseguir el máximo rendimiento de las consultas en las tablas de almacén de columnas, es importante la calidad de los segmentos.  Escriben filas en las tablas de almacén de columnas bajo presión de memoria afecta a la calidad de segmento.  La calidad de segmento se puede medir por el número de filas de un grupo de filas comprimido.  Para obtener instrucciones detalladas acerca de la detección y mejora de la calidad de los segmentos en las tablas de almacén de columnas agrupadas, consulte la sección [Causas de una calidad deficiente del índice de almacén de columnas][Causes of poor columnstore index quality] del artículo [Indexación de tablas en SQL Data Warehouse][Table indexes].  Como es importante que los segmentos de almacén de columnas sean de una buena calidad, es conveniente usar identificadores de usuario que se encuentren en la clase de recursos grande o mediana para cargar los datos. El uso de [unidades de almacenamiento de datos](what-is-a-data-warehouse-unit-dwu-cdwu.md) inferiores significa que desea asignar una clase de recurso mayor para el usuario que realiza la carga.

Dado que las tablas de almacén de columnas generalmente no insertan datos en un segmento del almacén de columnas comprimido hasta que hay más de 1 millón de filas por tabla y cada tabla de SQL Data Warehouse se divide en 60 partes, como norma general, las tablas de almacén de columnas no serán útiles para las consultas a menos que la tabla tenga más de 60 millones de filas.  Para las tablas con menos de 60 millones de filas, podría no tener sentido el índice de almacén de columnas.  Pero tampoco molesta.  Además, si divide los datos, recuerde que cada parte deberá tener 1 millón de filas para beneficiarse de un índice de almacén de columnas agrupadas.  Si una tabla tiene 100 particiones, deberá tener al menos 6 mil millones de filas para beneficiarse del almacén de columnas agrupadas (60 distribuciones * 100 particiones * 1 millón de filas).  Si la tabla no tiene 6 mil millones de filas en este ejemplo, reduzca el número de particiones o considere la posibilidad de usar una tabla de apilamiento en su lugar.  También puede experimentar para ver si consigue un mejor rendimiento con una tabla de apilamiento con índices secundarios, en lugar de con una tabla de almacén de columnas.

Al consultar una tabla de almacén de columnas, las consultas se ejecutarán más rápido si selecciona solo las que necesita.  

Consulte también [Indexación de tablas en SQL Data Warehouse][Table indexes], [Guía de índices de almacén de columnas][Columnstore indexes guide] y [Regeneración de índices de almacén de columnas][Rebuilding columnstore indexes]

## <a name="use-larger-resource-class-to-improve-query-performance"></a>Uso de clases de recursos más grandes para mejorar el rendimiento de las consultas
SQL Data Warehouse usa grupos de recursos para asignar memoria a las consultas.  De manera predeterminada, todos los usuarios se asignan a los recursos de clase pequeña, que concede a 100 MB de memoria por distribución.  Dado que siempre hay 60 distribuciones y cada distribución tiene un mínimo de 100 MB, la asignación de memoria total del sistema es de 6 000 MB o justo por debajo de 6 GB.  Algunas consultas, como las combinaciones de gran tamaño o las cargas a las tablas de almacén de columnas agrupadas, se beneficiarán de las mayores asignaciones de memoria.  Algunas consultas, como los exámenes puros, no sufrirán cambios.  Por otro lado, usar las clases de recursos mayores afecta la simultaneidad, por lo que deberá tener esto en cuenta antes de cambiar todos los usuarios a una clase de recursos grande.

Consulte también [Clases de recursos para la administración de cargas de trabajo](resource-classes-for-workload-management.md)

## <a name="use-smaller-resource-class-to-increase-concurrency"></a>Menor clase de recursos para aumentar la simultaneidad
Si observa que las consultas de usuario se retrasan bastante, es posible que los usuarios se ejecutan en clases de recursos mayores y consuman muchas ranuras de simultaneidad, lo que pone en cola otras consultas.  Para ver si hay consultas de usuarios en cola, ejecute `SELECT * FROM sys.dm_pdw_waits` para ver si se devuelven filas.

Consulte también [Clases de recursos para la administración de cargas de trabajo](resource-classes-for-workload-management.md), [sys.dm_pdw_waits][sys.dm_pdw_waits]

## <a name="use-dmvs-to-monitor-and-optimize-your-queries"></a>Uso de vistas de administración dinámica (DMV) para supervisar y optimizar las consultas
SQL Data Warehouse tiene varias DMV que sirven para supervisar la ejecución de la consulta.  El siguiente artículo de supervisión le guía con instrucciones paso a paso acerca de cómo ver los detalles de una consulta en curso.  Usar la opción LABEL con las consultas puede ayudar a encontrar rápidamente las consultas en estas DMV.

Consulte también [Supervisión de la carga de trabajo mediante DMV][Monitor your workload using DMVs], [LABEL][LABEL], [OPTION][OPTION], [sys.dm_exec_sessions][sys.dm_exec_sessions], [sys.dm_pdw_exec_requests][sys.dm_pdw_exec_requests], [sys.dm_pdw_request_steps][sys.dm_pdw_request_steps], [sys.dm_pdw_sql_requests][sys.dm_pdw_sql_requests], [sys.dm_pdw_dms_workers], [DBCC PDW_SHOWEXECUTIONPLAN][DBCC PDW_SHOWEXECUTIONPLAN], [sys.dm_pdw_waits][sys.dm_pdw_waits]

## <a name="other-resources"></a>Otros recursos:
Consulte también el artículo [Solución de problemas de Azure SQL Data Warehouse][Troubleshooting] para conocer los problemas comunes y sus soluciones.

Si no encuentra lo que buscaba en este artículo, pruebe a usar la "búsqueda de documentos" en la parte izquierda de esta página para buscar todos los documentos de Azure SQL Data Warehouse.  El [foro de Azure SQL Data Warehouse][Azure SQL Data Warehouse MSDN Forum] es un lugar para formular preguntas a otros usuarios y al grupo de productos de SQL Data Warehouse.  Supervisamos continuamente este foro para garantizar que sus preguntas las responde otro usuario o alguno de nosotros.  Si prefiere formular sus preguntas en Stack Overflow, también tenemos un [foro de Stack Overflow acerca de Azure SQL Data Warehouse][Azure SQL Data Warehouse Stack Overflow Forum].

Por último, use la página [de comentarios de Azure SQL Data Warehouse][Azure SQL Data Warehouse Feedback] para realizar solicitudes de características.  Añadir sus solicitudes o valoraciones positivas sobre otras solicitudes realmente nos ayuda a priorizar las características.

<!--Image references-->

<!--Article references-->
[Create a support ticket]: ./sql-data-warehouse-get-started-create-support-ticket.md
[Create table as select (CTAS)]: ./sql-data-warehouse-develop-ctas.md
[Table overview]: ./sql-data-warehouse-tables-overview.md
[Table data types]: ./sql-data-warehouse-tables-data-types.md
[Table distribution]: ./sql-data-warehouse-tables-distribute.md
[Table indexes]: ./sql-data-warehouse-tables-index.md
[Causes of poor columnstore index quality]: ./sql-data-warehouse-tables-index.md#causes-of-poor-columnstore-index-quality
[Rebuilding columnstore indexes]: ./sql-data-warehouse-tables-index.md#rebuilding-indexes-to-improve-segment-quality
[Table partitioning]: ./sql-data-warehouse-tables-partition.md
[Manage table statistics]: ./sql-data-warehouse-tables-statistics.md
[Temporary tables]: ./sql-data-warehouse-tables-temporary.md
[Guide for using PolyBase]: ./guidance-for-loading-data.md
[Load data]: ./design-elt-data-loading.md
[Move data with Azure Data Factory]: ../data-factory/transform-data-using-machine-learning.md
[Load data with Azure Data Factory]: ../data-factory/load-azure-sql-data-warehouse.md
[Load data with bcp]: /sql/tools/bcp-utility
[Load data with PolyBase]: ./load-data-wideworldimportersdw.md
[Monitor your workload using DMVs]: ./sql-data-warehouse-manage-monitor.md
[Pause compute resources]: ./sql-data-warehouse-manage-compute-overview.md#pause-compute-bk
[Resume compute resources]: ./sql-data-warehouse-manage-compute-overview.md#resume-compute-bk
[Scale compute resources]: ./sql-data-warehouse-manage-compute-overview.md#scale-compute
[Understanding transactions]: ./sql-data-warehouse-develop-transactions.md
[Optimizing transactions]: ./sql-data-warehouse-develop-best-practices-transactions.md
[Troubleshooting]: ./sql-data-warehouse-troubleshoot.md
[LABEL]: ./sql-data-warehouse-develop-label.md

<!--MSDN references-->
[ALTER TABLE]: https://msdn.microsoft.com/library/ms190273.aspx
[CREATE EXTERNAL FILE FORMAT]: https://msdn.microsoft.com/library/dn935026.aspx
[CREATE STATISTICS]: https://msdn.microsoft.com/library/ms188038.aspx
[CREATE TABLE]: https://msdn.microsoft.com/library/mt203953.aspx
[CREATE TABLE AS SELECT]: https://msdn.microsoft.com/library/mt204041.aspx
[DBCC PDW_SHOWEXECUTIONPLAN]: https://msdn.microsoft.com/library/mt204017.aspx
[INSERT]: https://msdn.microsoft.com/library/ms174335.aspx
[OPTION]: https://msdn.microsoft.com/library/ms190322.aspx
[TRUNCATE TABLE]: https://msdn.microsoft.com/library/ms177570.aspx
[UPDATE STATISTICS]: https://msdn.microsoft.com/library/ms187348.aspx
[sys.dm_exec_sessions]: https://msdn.microsoft.com/library/ms176013.aspx
[sys.dm_pdw_exec_requests]: https://msdn.microsoft.com/library/mt203887.aspx
[sys.dm_pdw_request_steps]: https://msdn.microsoft.com/library/mt203913.aspx
[sys.dm_pdw_sql_requests]: https://msdn.microsoft.com/library/mt203889.aspx
[sys.dm_pdw_dms_workers]: https://msdn.microsoft.com/library/mt203878.aspx
[sys.dm_pdw_waits]: https://msdn.microsoft.com/library/mt203893.aspx
[Columnstore indexes guide]: https://msdn.microsoft.com/library/gg492088.aspx

<!--Other Web references-->
[Selecting table distribution]: https://blogs.msdn.microsoft.com/sqlcat/20../../choosing-hash-distributed-table-vs-round-robin-distributed-table-in-azure-sql-dw-service/
[Azure SQL Data Warehouse Feedback]: https://feedback.azure.com/forums/307516-sql-data-warehouse
[Azure SQL Data Warehouse MSDN Forum]: https://social.msdn.microsoft.com/Forums/sqlserver/home?forum=AzureSQLDataWarehouse
[Azure SQL Data Warehouse Stack Overflow Forum]:  https://stackoverflow.com/questions/tagged/azure-sqldw
[Azure SQL Data Warehouse loading patterns and strategies]: https://blogs.msdn.microsoft.com/sqlcat/20../../azure-sql-data-warehouse-loading-patterns-and-strategies/
