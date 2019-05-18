---
title: 'Límites de capacidad: Azure SQL Data Warehouse | Microsoft Docs'
description: Valores máximos permitidos para los distintos componentes de Azure SQL Data Warehouse.
services: sql-data-warehouse
author: happynicolle
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.subservice: design
ms.date: 11/14/2018
ms.author: nicw
ms.reviewer: igorstan
ms.openlocfilehash: f3c2ecbb4c83132b674b4c296adc1339027f5215
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65797793"
---
# <a name="sql-data-warehouse-capacity-limits"></a>Límites de capacidad de SQL Data Warehouse
Valores máximos permitidos para los distintos componentes de Azure SQL Data Warehouse.

## <a name="workload-management"></a>Administración de cargas de trabajo
| Categoría | DESCRIPCIÓN | Máximo |
|:--- |:--- |:--- |
| [Unidades de almacenamiento de datos (DWU)](what-is-a-data-warehouse-unit-dwu-cdwu.md) |Máximo de DWU para una sola instancia de SQL Data Warehouse | Gen1: DW6000<br></br>Gen2: DW30000c |
| [Unidades de almacenamiento de datos (DWU)](what-is-a-data-warehouse-unit-dwu-cdwu.md) |Valor predeterminado de la DTU por servidor |54 000<br></br>De forma predeterminada, cada servidor SQL (por ejemplo, myserver.database.windows.net) tiene una cuota de DTU de 54 000, lo que permite un máximo de DW6000c. Esta cuota es simplemente un límite de seguridad. Puede aumentar su cuota mediante la [creación de una incidencia de soporte técnico](sql-data-warehouse-get-started-create-support-ticket.md) y la selección de *Cuota* como el tipo de solicitud.  Para calcular las necesidades de la DTU, multiplique 7,5 por el total de DWU necesarias o multiplique 9,0 por el total de cDWU necesarias. Por ejemplo:<br></br>DW6000 x 7,5 = 45 000 DTU<br></br>DW6000c x 9,0 = 54 000 DTU<br></br>Puede ver el consumo de DTU actual en la opción de SQL Server en el portal. Tanto las bases de datos en pausa como las no pausadas cuentan en la cuota de DTU. |
| Conexión de base de datos |Máximo simultáneas sesiones abiertas |1024<br/><br/>El número de sesiones abiertas simultáneas variará en función de la unidad seleccionada. DWU600c y versiones posteriores admiten un máximo de 1024 sesiones abiertas. DWU500c y a continuación, admitir un límite máximo de la sesión abierta simultáneas de 512. Tenga en cuenta que no hay límite en el número de consultas que se pueden ejecutar a la vez. Cuando se supera el límite de simultaneidad, la solicitud entra en una cola interna donde espera para su proceso. |
| Conexión de base de datos |Memoria máxima para instrucciones preparadas |20 MB |
| [Administración de cargas de trabajo](resource-classes-for-workload-management.md) |N.º máximo de consultas simultáneas |128<br/><br/> SQL Data Warehouse puede ejecutar 128 consultas simultáneas como máximo y pone en cola las restantes.<br/><br/>El número de consultas simultáneas se puede reducir cuando los usuarios se asignan a clases de recursos superiores o cuando SQL Data Warehouse tiene una configuración de [unidad de almacenamiento de datos](memory-and-concurrency-limits.md) inferior. Algunas consultas, como las consultas DMV, siempre se pueden ejecutar y no afectan al límite de consultas simultáneas. Para más detalles sobre la ejecución de consultas simultáneas, consulte el artículo sobre [valores máximos de simultaneidad](memory-and-concurrency-limits.md#concurrency-maximums). |
| [tempdb](sql-data-warehouse-tables-temporary.md) |GB máximos: |399 GB por DW100. Por lo tanto, en DWU1000 el tamaño de tempdb es de 3,99 TB. |

## <a name="database-objects"></a>Objetos de base de datos
| Categoría | DESCRIPCIÓN | Máximo |
|:--- |:--- |:--- |
| Base de datos |Tamaño máx. | Gen1: 240 TB comprimidos en disco. Este espacio es independiente del espacio de tempdb o de registro y, por tanto, está dedicado a tablas permanentes.  La compresión del almacén de columnas en clúster se estima en 5X.  Esta compresión permite que la base de datos crezca a aproximadamente 1 PB cuando todas las tablas tienen el almacén de columnas en clúster (el tipo de tabla predeterminada). <br/><br/> Gen2: 240 TB para el almacén de filas y almacenamiento ilimitado para las tablas de almacén de columnas |
| Tabla |Tamaño máx. |60 TB comprimidos en disco |
| Tabla |Tablas por base de datos | 100 000 |
| Tabla |Columnas por tabla |1024 columnas |
| Tabla |Bytes por columna |Depende de la columna de [tipo de datos](sql-data-warehouse-tables-data-types.md). El límite es 8000 para los tipos de datos char, 4000 para nvarchar o 2 GB para los tipos de datos MAX. |
| Tabla |Bytes por fila, tamaño definido |8060 bytes<br/><br/>El número de bytes por fila se calcula de la misma forma que para SQL Server con la compresión de página. Al igual que SQL Server, SQL Data Warehouse admite el almacenamiento con desbordamiento de fila, lo que permite insertar **columnas de longitud variable** de forma no consecutiva. Cuando se insertan filas de longitud variable, solo se almacena la raíz de 24 bytes en el registro principal. Para obtener más información, consulte [Datos de desbordamiento de fila superiores a 8 KB](https://msdn.microsoft.com/library/ms186981.aspx). |
| Tabla |Particiones por tabla |15 000<br/><br/>Para obtener un alto rendimiento, se recomienda reducir al mínimo el número de particiones que necesita, pero sin perder de vista sus requisitos empresariales. A medida que crece el número de particiones, la sobrecarga de operaciones de lenguaje de definición de datos (DDL) y lenguaje de manipulación de datos (DML) crece y da lugar a un rendimiento más lento. |
| Tabla |Caracteres por valor de límite de partición |4000 |
| Índice |Índices no agrupados por tabla |50<br/><br/>Solo se aplica a tablas de almacén de filas. |
| Índice |Índices agrupados por tabla |1<br><br/>Se aplica a tablas de almacén de filas y de almacén de columnas. |
| Índice |Tamaño de clave de índice |900 bytes<br/><br/>Solo se aplica a los índices de almacén de filas.<br/><br/>Si los datos existentes en las columnas no superan los 900 bytes cuando se crea el índice, pueden crearse índices en columnas varchar con un tamaño máximo de más de 900 bytes. Sin embargo, las posteriores acciones INSERT o UPDATE en las columnas que hacen que el número total supere los 900 bytes darán error. |
| Índice |Columnas de clave por índice |16<br/><br/>Solo se aplica a los índices de almacén de filas. Los índices de almacén de columnas agrupados incluyen todas las columnas. |
| Estadísticas |Tamaño de los valores de columna combinados |900 bytes |
| Estadísticas |Columnas por objeto de estadísticas |32 |
| Estadísticas |Estadísticas creadas en columnas por tabla |30 000 |
| Procedimientos almacenados |Niveles máximos de anidamiento |8 |
| Vista |Columnas por vista |1024 |

## <a name="loads"></a>Cargas
| Categoría | DESCRIPCIÓN | Máximo |
|:--- |:--- |:--- |
| Cargas de PolyBase |MB por fila |1<br/><br/>Polybase carga las filas que son menores que 1 MB. No se admite la carga de tipos de datos LOB en tablas con un índice de almacén de columnas en clúster (CCI).<br/><br/> |

## <a name="queries"></a>Consultas
| Categoría | DESCRIPCIÓN | Máximo |
|:--- |:--- |:--- |
| Consultar |Consultas en cola en tablas de usuario |1000 |
| Consultar |Consultas simultáneas en vistas de sistema |100 |
| Consultar |Consultas en cola en vistas de sistema |1000 |
| Consultar |Parámetros máximos |2098 |
| Lote |Tamaño máximo |65 536*4096 |
| Resultados de SELECT |Columnas por fila |4096<br/><br/>Nunca se pueden tener más de 4096 columnas por fila en el resultado de SELECT. No hay ninguna garantía de que pueda tener siempre 4096. Si el plan de consulta requiere una tabla temporal, se podría aplicar el máximo de 1024 columnas por tabla. |
| SELECCIONAR |Subconsultas anidadas |32<br/><br/>Nunca se pueden tener más de 32 subconsultas anidadas en una instrucción SELECT. No hay ninguna garantía de que siempre pueda tener 32. Por ejemplo, una instrucción JOIN puede introducir una subconsulta en el plan de consulta. El número de subconsultas también puede estar limitado por la memoria disponible. |
| SELECCIONAR |Columnas por JOIN |1024 columnas<br/><br/>Nunca se pueden tener más de 1024 columnas en la instrucción JOIN. No hay ninguna garantía de que siempre pueda tener 1024. Si el plan JOIN requiere una tabla temporal con más columnas que el resultado de JOIN, se aplica el límite de 1024 a la tabla temporal. |
| SELECCIONAR |Bytes por columnas GROUP BY |8060<br/><br/>Las columnas de la cláusula GROUP BY pueden tener como máximo 8060 bytes. |
| SELECCIONAR |Bytes por columnas ORDER BY |8060 bytes<br/><br/>Las columnas de la cláusula ORDER BY pueden tener como máximo 8060 bytes |
| Identificadores por instrucción |Número de identificadores de referencia |65 535<br/><br/>SQL Data Warehouse limita el número de identificadores que pueden incluirse en una única expresión de una consulta. Si se supera este número se produce el error de SQL Server 8632. Para obtener más información, consulte el tema [Error interno: se ha alcanzado un límite de servicios de expresión](https://support.microsoft.com/en-us/help/913050/error-message-when-you-run-a-query-in-sql-server-2005-internal-error-a). |
| Literales de cadena | Número de literales de cadena en una instrucción | 20.000 <br/><br/>SQL Data Warehouse limita el número de constantes de cadena de una única expresión de una consulta. Si se supera este número se produce el error de SQL Server 8632.|

## <a name="metadata"></a>Metadatos
| Vista de sistema | Número máximo de filas |
|:--- |:--- |
| sys.dm_pdw_component_health_alerts |10 000 |
| sys.dm_pdw_dms_cores |100 |
| sys.dm_pdw_dms_workers |Número total de trabajadores de DMS para las 1000 solicitudes de SQL más recientes. |
| sys.dm_pdw_errors |10 000 |
| sys.dm_pdw_exec_requests |10 000 |
| sys.dm_pdw_exec_sessions |10 000 |
| sys.dm_pdw_request_steps |Número total de pasos para las 1000 solicitudes SQL más recientes que se almacenan en sys.dm_pdw_exec_requests. |
| sys.dm_pdw_os_event_logs |10 000 |
| sys.dm_pdw_sql_requests |Las 1000 solicitudes SQL más recientes que se almacenan en sys.dm_pdw_exec_requests. |

## <a name="next-steps"></a>Pasos siguientes
Para obtener recomendaciones sobre el uso de SQL Data Warehouse, consulte la [hoja de referencia rápida](cheat-sheet.md).
