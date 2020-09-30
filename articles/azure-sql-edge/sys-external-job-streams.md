---
title: 'sys.external_job_streams (Transact-SQL): Azure SQL Edge'
description: Obtenga información sobre el uso de sys.external_job_streams en Azure SQL Edge.
keywords: sys.external_job_streams, SQL Edge
services: sql-edge
ms.service: sql-edge
ms.topic: reference
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 05/19/2019
ms.openlocfilehash: 35010d3aba7f6d5ee3185291c917ff7726ba8bd7
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90900351"
---
# <a name="sysexternal_job_streams-transact-sql"></a>sys.external_job_streams (Transact-SQL)

Devuelve una fila para el objeto de flujo externo de entrada o salida asignado a un trabajo de streaming externo.

|Nombre de la columna|Tipo de datos|Descripción|  
|-----------------|---------------|-----------------|
|**job_id**|**int**| Número de identificación del objeto del trabajo de streaming. Esta columna se asigna a la columna object_id de sys.external_streaming_jobs.|
|**stream_id**|**int**| Número de identificación del objeto del flujo. Esta columna se asigna a la columna object_id de sys.external_streams. |
|**is_input**|**bit**| 1 si el objeto del flujo se usa como entrada para el trabajo de streaming; en caso contrario, 0.|
|**is_output**|**bit**| 1 si el objeto del flujo se usa como salida para el trabajo de streaming; en caso contrario, 0.|

## <a name="example"></a>Ejemplo

Esta vista de catálogo se utiliza junto con las vistas de catálogo `sys.external_streams` y `sys.external_streaming_jobs`. A continuación, se muestra una consulta de ejemplo.

```sql
Select
    sj.Name as Job_Name,
    sj.Create_date as Job_Create_date,
    sj.modify_date as Job_Modify_date,
    sj.statement as Stream_Job_Query,
    Input_Stream_Name =
        Case js.is_input
            when 1 then s.Name
            else null
        END,
    output_Stream_Name =
        case js.is_output
            when 1 then s.Name
            else null
        END,
    s.location as Stream_Location
from sys.external_job_streams js
inner join sys.external_streams s on s.object_id = js.stream_id
inner join sys.external_streaming_jobs sj on sj.object_id = js.job_id
```

## <a name="permissions"></a>Permisos

La visibilidad de los metadatos en las vistas de catálogo se limita a los elementos protegibles y que son propiedad de un usuario o sobre los que el usuario tiene algún permiso. Para obtener más información, consulte [Metadata Visibility Configuration](/sql/relational-databases/security/metadata-visibility-configuration/).

## <a name="see-also"></a>Consulte también

- [Vistas de catálogo (Transact-SQL)](/sql/relational-databases/system-catalog-views/catalog-views-transact-sql/)
- [Vistas del sistema (Transact-SQL)](/sql/t-sql/language-reference/)
