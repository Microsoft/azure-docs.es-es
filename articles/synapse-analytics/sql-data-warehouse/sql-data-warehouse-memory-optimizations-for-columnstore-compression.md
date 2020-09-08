---
title: Mejora en el rendimiento del índice de almacén de columnas
description: Reduzca los requisitos de memoria o aumente la memoria disponible para maximizar el número de filas en cada grupo de filas.
services: synapse-analytics
author: kevinvngo
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 03/22/2019
ms.author: kevin
ms.reviewer: igorstan
ms.custom: azure-synapse
ms.openlocfilehash: 85a9c758f46150c422b55c6ac5cf7e62a429c74f
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88797775"
---
# <a name="maximizing-rowgroup-quality-for-columnstore"></a>Maximización de la calidad del grupo de filas del almacén de columnas

El número de filas de un grupo de filas determina la calidad del grupo de filas. Aumentar la memoria disponible puede maximizar el número de filas que un índice de almacén de columnas comprime en cada grupo de filas.  Emplee estos métodos para mejorar las tasas de compresión y el rendimiento de las consultas de los índices de almacén de columnas.

## <a name="why-the-rowgroup-size-matters"></a>Por qué importa el tamaño del grupo de filas

Como los índices de almacén de columnas examinan una tabla mediante el examen de segmentos de columna de grupos de filas individuales, al maximizar el número de filas de cada grupo de estas, se mejora el rendimiento de las consultas.

Cuando los grupos de filas presentan un gran número de filas, la compresión de datos mejora; es decir, hay menos datos que se deben leer en el disco.

Para obtener más información sobre los grupos de filas, consulte [Guía de índices de almacén de columnas](/sql/relational-databases/indexes/columnstore-indexes-overview?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

## <a name="target-size-for-rowgroups"></a>Tamaño objetivo de los grupos de filas

Para obtener el mejor rendimiento de consultas, el objetivo consiste en maximizar el número de filas por grupo de un índice de almacén de columnas. Un grupo de filas puede tener un máximo de 1.048.576 filas.

No pasa nada si no se tiene el máximo número de filas por grupo de filas. Los índices de almacén de columnas logran un buen rendimiento cuando los grupos de filas cuentan con al menos 100.000 filas.

## <a name="rowgroups-can-get-trimmed-during-compression"></a>Los grupos de filas pueden recortarse durante la compresión

Durante una carga masiva o regeneración de índice de almacén columnas, a veces no queda suficiente memoria disponible para comprimir todas las filas designadas para cada grupo de filas. Cuando existe una presión de memoria, los índices de almacén de columnas recortan el tamaño de los grupos de filas para que se pueda realizar la compresión en el almacén de columnas.

Cuando no hay memoria suficiente para comprimir al menos 10 000 filas en cada grupo de filas, se genera un error.

Para obtener más información sobre la carga masiva, consulte [Carga de datos en un índice de almacén de columnas agrupado](/sql/relational-databases/indexes/columnstore-indexes-data-loading-guidance?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

## <a name="how-to-monitor-rowgroup-quality"></a>Cómo supervisar la calidad del grupo de filas

La vista de administración dinámica sys.dm_pdw_nodes_db_column_store_row_group_physical_stats ([sys.dm_db_column_store_row_group_physical_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-column-store-row-group-physical-stats-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) contiene la definición de vista que coincide con SQL Database) que expone información útil, como el número de filas en los grupos de filas y el motivo del recorte (si es que se recortó).

Puede crear la siguiente vista como una forma práctica para consultar esta DMV a fin de obtener información sobre el recorte del grupo de filas.

```sql
create view dbo.vCS_rg_physical_stats
as
with cte
as
(
select   tb.[name]                    AS [logical_table_name]
,        rg.[row_group_id]            AS [row_group_id]
,        rg.[state]                   AS [state]
,        rg.[state_desc]              AS [state_desc]
,        rg.[total_rows]              AS [total_rows]
,        rg.[trim_reason_desc]        AS trim_reason_desc
,        mp.[physical_name]           AS physical_name
FROM    sys.[schemas] sm
JOIN    sys.[tables] tb               ON  sm.[schema_id]          = tb.[schema_id]
JOIN    sys.[pdw_table_mappings] mp   ON  tb.[object_id]          = mp.[object_id]
JOIN    sys.[pdw_nodes_tables] nt     ON  nt.[name]               = mp.[physical_name]
JOIN    sys.[dm_pdw_nodes_db_column_store_row_group_physical_stats] rg      ON  rg.[object_id]     = nt.[object_id]
                                                                            AND rg.[pdw_node_id]   = nt.[pdw_node_id]
                                        AND rg.[distribution_id]    = nt.[distribution_id]
)
select *
from cte;
```

>[!TIP]
> Para mejorar el rendimiento de Synapse SQL, considere la posibilidad de usar **sys.pdw_permanent_table_mappings** en lugar de **sys.pdw_table_mappings** en tablas de usuario permanentes. Para más información, consulte **[sys.pdw_permanent_table_mappings &#40;Transact-SQL&#41;](/sql/relational-databases/system-catalog-views/sys-pdw-permanent-table-mappings-transact-sql?view=azure-sqldw-latest)** .

trim_reason_desc indica si el grupo de filas se ha recortado (trim_reason_desc = NO_TRIM implica que no ha habido ningún recorte y que el grupo de filas es de calidad óptima). Los siguientes motivos de recorte indican el recorte prematuro del grupo de filas:

- BULKLOAD: Este motivo de recorte se usa cuando el lote entrante de filas de la carga tenía menos de 1 millón de filas. El motor creará grupos de filas comprimidos si hay más de 100.000 filas que se van a insertar (en lugar de insertar en el almacén delta), pero establece el motivo de recorte en BULKLOAD. En este escenario, considere la posibilidad de aumentar la carga por lotes para incluir más filas. Además, vuelva a evaluar el esquema de partición para asegurarse de que no es demasiado granular si los grupos de filas no pueden abarcar los límites de partición.
- MEMORY_LIMITATION: Para crear grupos de filas con 1 millón de filas, el motor necesita una determinada cantidad de memoria de trabajo. Cuando la memoria disponible de la sesión de carga es inferior a la memoria de trabajo necesaria, los grupos de filas se recortan prematuramente. En las siguientes secciones se explica cómo estimar la memoria necesaria y asignar más.
- DICTIONARY_SIZE: Este motivo de recorte indica que el recorte del grupo de filas se ha producido porque había al menos una columna de cadena con cadenas de cardinalidad anchas o altas. El tamaño del diccionario está limitado a 16 MB de memoria y, cuando se alcanza este límite, se comprime el grupo de filas. Si ejecuta en esta situación, considere la posibilidad de aislar la columna problemática en una tabla independiente.

## <a name="how-to-estimate-memory-requirements"></a>Cómo calcular los requisitos de memoria

<!--
To view an estimate of the memory requirements to compress a rowgroup of maximum size into a columnstore index, download and run the view [dbo.vCS_mon_mem_grant](). This view shows the size of the memory grant that a rowgroup requires for compression in to the columnstore.
-->

La memoria máxima requerida para comprimir un grupo de filas es aproximadamente:

- 72 MB +
- \#filas \*\#columnas \* 8 bytes +
- \#filas \*\#columnas de cadena corta \* 32 bytes +
- \#columnas de cadena larga \* 16 MB para el diccionario de compresión

> [!NOTE]
> Las columnas de cadena corta emplean tipos de datos de cadena de ≤32 bytes y las de cadena larga usan tipos de datos de cadena de >32 bytes.

Las cadenas largas que se comprimen con un método de compresión diseñado para comprimir texto. Este método de compresión utiliza un *diccionario* para almacenar patrones de texto. El tamaño máximo de un diccionario es de 16 MB. Hay solo un diccionario para cada columna de cadena larga del grupo de filas.

Para ver una discusión en profundidad sobre los requisitos de memoria del almacén de columnas, eche un vistazo al vídeo [Escalado de grupo de SQL de Synapse: configuración y directrices](https://channel9.msdn.com/Events/Ignite/2016/BRK3291).

## <a name="ways-to-reduce-memory-requirements"></a>Formas de reducir los requisitos de memoria

Ponga en práctica las siguientes técnicas a fin de reducir los requisitos de memoria para comprimir los grupos de filas en índices de almacén de columnas.

### <a name="use-fewer-columns"></a>Utilizar menos columnas

Si es posible, diseñe la tabla con menos columnas. Cuando se comprime un grupo de filas en el almacén de columnas, el índice de este comprime cada segmento de columna por separado.

Por lo tanto, los requisitos de memoria para comprimir un grupo de filas aumentan a la par que el número de columnas.

### <a name="use-fewer-string-columns"></a>Utilizar menos columnas de cadena

Las columnas de tipos de datos de cadena requieren más memoria que los tipos de datos numéricos y de fecha. Para reducir los requisitos de memoria, plantéese la posibilidad de quitar las columnas de cadena de las tablas de hechos y colocarlos en las tablas de dimensiones más pequeñas.

Requisitos de memoria adicionales para la compresión de cadenas:

- Los tipos de datos de cadena de hasta 32 caracteres pueden requerir 32 bytes adicionales por valor.
- Los tipos de datos de cadena con más de 32 caracteres se comprimen con métodos de diccionario.  Cada columna del grupo de filas puede requerir hasta 16 MB adicionales para crear el diccionario.

### <a name="avoid-over-partitioning"></a>Evitar un exceso de particiones

Los índices de almacén de columnas crean uno o varios grupos de filas por partición. Para al grupo de SQL en Azure Synapse Analytics, el número de particiones aumenta rápidamente porque los datos se distribuyen y cada distribución se particiona.

Si la tabla presenta demasiadas particiones, es posible que no haya un número de filas suficiente para llenar los grupos de filas. La falta de filas no crea presión de memoria durante la compresión. Pero genera grupos de filas que no logran el mejor rendimiento de las consultas del almacén de columnas.

Otro motivo para evitar el exceso de particiones es que cargar filas en un índice de columnas en una tabla particionada conlleva una sobrecarga para la memoria.

Durante la carga, muchas particiones podrían recibir las filas entrantes, que se guardan en la memoria hasta que cada partición alcance el número de filas suficiente para llevar a cabo la compresión. Si se cuenta con demasiadas particiones, se genera una presión de memoria adicional.

### <a name="simplify-the-load-query"></a>Simplificar la consulta de carga

La base de datos comparte la concesión de memoria para una consulta con todos los operadores de esta. Cuando una consulta de carga presenta ordenaciones y uniones complejas, se reduce la memoria disponible para la compresión.

Diseñe la consulta de carga para que se centre únicamente en cargar la consulta. Si tiene que ejecutar transformaciones en los datos, ejecútelas de forma independiente de la consulta de carga. Por ejemplo, almacene provisionalmente los datos en una tabla de montón, ejecute las transformaciones y, después, cargue la tabla de almacenamiento provisional en el índice de almacén de columnas.

> [!TIP]
> Puede cargar los datos primero y, después, usar el sistema MPP para transformar los datos.

### <a name="adjust-maxdop"></a>Ajustar MAXDOP

Cada distribución comprime los grupos de filas en el almacén de columnas en paralelo cuando hay más de un núcleo de CPU disponible por distribución.

El paralelismo requiere recursos de memoria adicionales, lo que puede generar presión de memoria y recorte de los grupos de filas.

Para reducir la presión de memoria, puede usar la sugerencia de consulta MAXDOP a fin de forzar la operación de carga con el objetivo de que se ejecute en modo serie dentro de cada distribución.

```sql
CREATE TABLE MyFactSalesQuota
WITH (DISTRIBUTION = ROUND_ROBIN)
AS SELECT * FROM FactSalesQuota
OPTION (MAXDOP 1);
```

## <a name="ways-to-allocate-more-memory"></a>Formas de asignar más memoria

Juntos, el tamaño de DWU y la clase de recursos de usuario, determinan cuánta memoria hay disponible para la consulta de un usuario.

A fin de incrementar la concesión de memoria para una consulta de carga, puede aumentar el número de DWU o la clase de recursos.

- Para aumentar las DWU, consulte [¿Cómo se realiza el escalado del rendimiento?](quickstart-scale-compute-portal.md)
- Para cambiar la clase de recursos de una consulta, consulte [Cambio de ejemplo de clase de recursos de usuario](resource-classes-for-workload-management.md#change-a-users-resource-class).

## <a name="next-steps"></a>Pasos siguientes

Para descubrir más formas de mejorar el rendimiento del grupo de SQL, consulte el tema de [información general sobre rendimiento](cheat-sheet.md).
