---
title: 'Administración de datos históricos con directivas de retención: Azure SQL Edge'
description: Aprenda a administrar datos históricos con directivas de retención en Azure SQL Edge.
keywords: SQL Edge, retención de datos
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/04/2020
ms.openlocfilehash: 45ce874ffb626f63b2239c66afdefd091114cbd2
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90888139"
---
# <a name="manage-historical-data-with-retention-policy"></a>Administración de datos históricos con directivas de retención

La retención de datos puede habilitarse en la base de datos y en cualquiera de las tablas subyacentes de forma individual, lo que permite a los usuarios crear directivas de vencimiento flexibles para sus tablas y bases de datos. Aplicar la retención de datos es muy sencillo: solo es necesario establecer un parámetro al crear la tabla o como parte de una operación posterior a su creación. 

Después de definir la directiva de retención de datos para una base de datos y la tabla subyacente, se ejecuta una tarea de temporizador en segundo plano para quitar los registros obsoletos de la tabla habilitada para la retención de datos. La identificación de las filas coincidentes y su eliminación de la tabla se producen de forma transparente, en la tarea en segundo plano que programa y ejecuta el sistema. La condición de vencimiento de las filas de la tabla se comprueba en función de la columna usada como columna de filtro (`filter_column`) en la definición de la tabla. Si el período de retención se establece, por ejemplo, en una semana, las filas de la tabla aptas para la limpieza cumplen una de las siguientes condiciones: 

- Si la columna de filtro usa el tipo de datos DATETIMEOFFSET, la condición es `filter_column < DATEADD(WEEK, -1, SYSUTCDATETIME())`.
- En caso contrario, la condición es `filter_column < DATEADD(WEEK, -1, SYSDATETIME())`.

## <a name="data-retention-cleanup-phases"></a>Fases de la limpieza de la retención de datos

La operación de limpieza de la retención de datos consta de dos fases. 
- Fase de detección: en esta fase, la operación de limpieza identifica todas las tablas de las bases de datos del usuario para crear una lista para la limpieza. La detección se ejecuta una vez al día.
- Fase de limpieza: en esta fase, se ejecuta la limpieza en todas las tablas con retención de datos finita, identificadas en la fase de detección. Si no se puede realizar la operación de limpieza en una tabla, esa tabla se omite en la ejecución actual y se reintentará su limpieza en la siguiente iteración. Durante la limpieza se aplican los siguientes principios:
    - Si una fila obsoleta está bloqueada por otra transacción, se omite. 
    - La limpieza se ejecuta, de forma predeterminada, con un tiempo de expiración de bloqueo de cinco segundos. Si los bloqueos no se pueden adquirir en una tabla durante el tiempo de espera, la tabla se omite en la ejecución actual y se incluirá en la siguiente iteración.
    - Si se produce un error durante la limpieza de una tabla, esa tabla se omite y se vuelve a incluir en la siguiente iteración.

## <a name="manual-cleanup"></a>Limpieza manual

En función de la configuración de retención de datos de una tabla y según la naturaleza de la carga de trabajo en la base de datos, es posible que el subproceso de limpieza automática no quite completamente todas las filas obsoletas durante su ejecución. Para ayudar con esto y permitir que los usuarios quiten manualmente las filas obsoletas, se ha introducido el procedimiento almacenado `sys.sp_cleanup_data_retention` en Azure SQL Edge. 

Este procedimiento almacenado toma tres parámetros. 
    - Nombre de esquema: nombre del esquema propietario de la tabla. Es un parámetro obligatorio. 
    - Nombre de la tabla: nombre de la tabla para la que se ejecuta la limpieza manual. Es un parámetro obligatorio. 
    - rowcount (parámetro de salida): variable de salida. Devuelve el número de filas limpiadas por el procedimiento almacenado de limpieza manual. Se trata de un parámetro opcional. 

En el ejemplo siguiente se muestra la ejecución del procedimiento almacenado de limpieza manual para la tabla `dbo.data_retention_table`.

```sql
declare @rowcnt bigint 
EXEC sys.sp_cleanup_data_retention 'dbo', 'data_retention_table', @rowcnt output 
select @rowcnt 
```

## <a name="how-obsolete-rows-are-deleted"></a>Eliminación de las filas obsoletas

El proceso de limpieza depende del diseño del índice de la tabla. Se crea una tarea en segundo plano para realizar la limpieza de datos obsoletos en todas las tablas con un período de retención finito. La lógica de limpieza del índice de almacén de filas (árbol B o montón) elimina la fila obsoleta en fragmentos más pequeños (hasta 10 000), lo que reduce la presión en el registro de base de datos y el subsistema de E/S. A pesar de que la lógica de limpieza usa el índice de árbol B necesario, no se puede garantizar el orden de las eliminaciones de las filas más antiguas en relación con el período de retención. Por tanto, no hay ninguna dependencia en el orden de limpieza en sus aplicaciones.

La tarea de limpieza del almacén de columnas agrupadas quita grupos de filas completos de una sola vez (normalmente, cada uno contiene 1 millón de filas), lo que es muy eficaz, especialmente cuando los datos se generan y quedan obsoletos a un ritmo alto.

![Limpieza de la retención de datos](./media/data-retention-cleanup/data-retention-cleanup.png)

La excelente compresión de datos y la limpieza eficaz de la retención hacen que el índice de almacén de columnas agrupadas sea una elección perfecta en escenarios en los que la carga de trabajo genera rápidamente una gran cantidad de datos.

> [!Note]
> En el caso de los índices de árbol B y los montones, la retención de datos ejecuta una consulta de eliminación en las tablas subyacentes, lo que puede entrar en conflicto con los desencadenadores de eliminación de las tablas. Se recomienda quitar estos desencadenadores de eliminación de las tablas o no habilitar la retención de datos en las tablas con un desencadenador DML de eliminación.

## <a name="monitoring-data-retention-cleanup"></a>Supervisión de la limpieza de la retención de datos

Las operaciones de limpieza de directivas de retención de datos se pueden supervisar mediante eventos extendidos (XEvents) en Azure SQL Edge. Para obtener más información sobre los eventos extendidos, vea la [información general de XEvents](https://docs.microsoft.com/sql/relational-databases/extended-events/extended-events). 

Los seis eventos extendidos enumerados a continuación ayudan a realizar un seguimiento del estado de las operaciones de limpieza. 

| Nombre | Descripción |
|------| ------------|
| data_retention_task_started  | Se produce cuando se inicia una tarea en segundo plano para la limpieza de las tablas con directivas de retención. |
| data_retention_task_completed  | Se produce cuando finaliza una tarea en segundo plano para la limpieza de las tablas con directivas de retención. |
| data_retention_task_exception  | Se produce cuando no se puede efectuar una tarea en segundo plano para la limpieza de las tablas con directivas de retención fuera del proceso de limpieza de retención específico de la tabla. |
| data_retention_cleanup_started  | Se produce cuando se inicia el proceso de limpieza de una tabla con directiva de retención de datos. |
| data_retention_cleanup_exception  | Se produce cuando no se puede efectuar el proceso de limpieza de una tabla con directiva de retención. |
| data_retention_cleanup_completed  | Se produce cuando finaliza el proceso de limpieza de una tabla con directiva de retención de datos. |  

Además, se ha agregado un nuevo tipo de búfer de anillo denominado `RING_BUFFER_DATA_RETENTION_CLEANUP` a la vista de administración dinámica sys.dm_os_ring_buffers. Esta vista se puede usar para supervisar las operaciones de limpieza de la retención de datos. 


## <a name="next-steps"></a>Pasos a seguir
- [Directiva de retención de datos](data-retention-overview.md)
- [Habilitación y deshabilitación de directivas de retención de datos](data-retention-enable-disable.md)
