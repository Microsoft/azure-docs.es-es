---
title: Ajuste del rendimiento con la copia en caché del conjunto de resultados
description: Información general sobre la característica de almacenamiento en caché del conjunto de resultados para grupos de SQL de Synapse en Azure Synapse Analytics
services: synapse-analytics
author: XiaoyuMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 10/10/2019
ms.author: xiaoyul
ms.reviewer: nidejaco;
ms.custom: azure-synapse
ms.openlocfilehash: 2c29ad48e8dc626617786e519e84c92875bc6170
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89378532"
---
# <a name="performance-tuning-with-result-set-caching"></a>Ajuste del rendimiento con la copia en caché del conjunto de resultados

Cuando se habilita el almacenamiento en caché del conjunto de resultados, Synapse SQL almacena automáticamente en caché los resultados de la consulta en la base de datos de usuario para usarlos de forma repetida.  Esto permite que las ejecuciones posteriores de la consulta obtengan los resultados directamente de la memoria caché persistente, por lo que no es necesario volver a realizar el proceso.   La copia en caché del conjunto de resultados mejora el rendimiento de las consultas y reduce la utilización de recursos de proceso.  Además, las consultas que usan conjuntos de resultados en la memoria caché no usan ningún espacio de simultaneidad y, por lo tanto, no cuentan para los límites de simultaneidad existentes. Por seguridad, los usuarios solo pueden acceder a los resultados en la memoria caché si tienen los mismos permisos de acceso a los datos que los usuarios que crearon estos resultados.  

## <a name="key-commands"></a>Comandos clave

[Activación o desactivación (ON/OFF) de la copia en caché del conjunto de resultados de una base de datos de usuario](/sql/t-sql/statements/alter-database-transact-sql-set-options?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)

[Activación o desactivación (ON/OFF) de la copia en caché del conjunto de resultados de una sesión](/sql/t-sql/statements/set-result-set-caching-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)

[Comprobación del tamaño del conjunto de resultados copiado en caché](/sql/t-sql/database-console-commands/dbcc-showresultcachespaceused-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)  

[Limpieza de la memoria caché](/sql/t-sql/database-console-commands/dbcc-dropresultsetcache-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)

## <a name="whats-not-cached"></a>No se copia en caché  

Una vez activada la copia en caché del conjunto de resultados de una base de datos, se copian en caché los resultados de todas las consultas hasta que la memoria caché está llena, excepto en el caso de las siguientes consultas:

- Consultas con funciones no deterministas como DateTime.Now()
- Consultas con funciones definidas por el usuario
- Consultas que usan tablas con la seguridad de nivel de fila o la seguridad de nivel de columna habilitadas
- Consultas que devuelven datos con un tamaño de fila superior a 64 kB
- Consultas que devuelven datos de gran tamaño (más de 10 GB) 

> [!IMPORTANT]
> Las operaciones para crear la caché del conjunto de resultados y recuperar datos de la caché se producen en el nodo de control de una instancia de grupo de SQL de Synapse.
> Cuando se activa el almacenamiento en caché de conjuntos de resultados, la ejecución de consultas que devuelven un conjunto de resultados grande (por ejemplo, > 1 GB) puede provocar una limitación elevada del nodo de control y ralentizar la respuesta de consulta general en la instancia.  Estas consultas se suelen usar durante la exploración de datos o las operaciones ETL. Para evitar el esfuerzo del nodo de control y los problemas de rendimiento, los usuarios deben desactivar el almacenamiento en caché de los conjuntos de resultados en la base de datos antes de ejecutar esos tipos de consultas.  

Ejecute esta consulta por el tiempo que tardan las operaciones de almacenamiento en caché del conjunto de resultados para una consulta:

```sql
SELECT step_index, operation_type, location_type, status, total_elapsed_time, command
FROM sys.dm_pdw_request_steps
WHERE request_id  = <'request_id'>;
```

Este es un ejemplo de una consulta ejecutada con el almacenamiento en caché de conjunto de resultados deshabilitado.

![Query-steps-with-rsc-disabled](./media/performance-tuning-result-set-caching/query-steps-with-rsc-disabled.png)

Este es un ejemplo de una consulta ejecutada con el almacenamiento en caché de conjunto de resultados habilitado.

![Query-steps-with-rsc-enabled](./media/performance-tuning-result-set-caching/query-steps-with-rsc-enabled.png)

## <a name="when-cached-results-are-used"></a>Uso de los resultados en la memoria caché

El conjunto de resultados en caché se reutiliza en una consulta si se cumplen todos los requisitos siguientes:

- El usuario que ejecuta la consulta tiene acceso a todas las tablas a las que se hace referencia en la consulta.
- Hay una coincidencia exacta entre la nueva consulta y la consulta anterior que generó la caché del conjunto de resultados.
- No hay cambios en los datos o en el esquema de las tablas a partir de las cuales se generó el conjunto de resultados en caché.

Ejecute este comando para comprobar si una consulta se ejecutó con un acierto o un error de caché de resultados. La columna result_cache_hit devuelve 1 para aciertos de caché, 0 para errores de caché y valores negativos para los motivos por los que no se ha usado el almacenamiento en caché del conjunto de resultados. Para más información, consulte [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

```sql
SELECT request_id, command, result_cache_hit FROM sys.dm_pdw_exec_requests
WHERE request_id = <'Your_Query_Request_ID'>
```

## <a name="manage-cached-results"></a>Administración de los resultados en caché

El tamaño máximo de la memoria caché del conjunto de resultados es de 1 TB por base de datos.  Los resultados en caché se invalidan automáticamente cuando cambian los datos de la consulta subyacente.  

Synapse SQL administra automáticamente la expulsión de la caché según la programación siguiente:

- Cada 48 horas si el conjunto de resultados no se ha usado o se ha invalidado.
- Cuando la memoria caché del conjunto de resultados se aproxima al tamaño máximo.

Los usuarios pueden vaciar manualmente toda la memoria caché del conjunto de resultados mediante una de estas opciones:

- Desactivación de la característica de copia en caché del conjunto de resultados de la base de datos
- Ejecución de DBCC DROPRESULTSETCACHE mientras está conectado a la base de datos

Al pausar una base de datos, no se vaciará el conjunto de resultados en caché.  

## <a name="next-steps"></a>Pasos siguientes

Para obtener más sugerencias sobre desarrollo, vea la [información general sobre desarrollo](sql-data-warehouse-overview-develop.md).
