---
title: 'sys.external_streaming_jobs (Transact-SQL): Azure SQL Edge'
description: Información sobre el uso de sys.external_streaming_jobs en Azure SQL Edge
keywords: sys.external_streaming_jobs, SQL Edge
services: sql-edge
ms.service: sql-edge
ms.topic: reference
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 05/19/2019
ms.openlocfilehash: 9643c58f5c9fa1db3e3eb7ec75ce6d3b41620aa3
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90900339"
---
# <a name="sysexternal_streaming_jobs-transact-sql"></a>sys.external_streaming_jobs (Transact-SQL)

Devuelve una fila por cada trabajo de streaming externo creado en el ámbito de la base de datos.

|Nombre de la columna|Tipo de datos|Descripción|  
|-----------------|---------------|-----------------|
|**name**|**sysname**|Nombre del flujo. Es único en la base de datos.|
|**object_id**|**int**|Número de identificación del objeto del flujo. Es único en la base de datos.|
|**principal_id**|**int**|Id. de la entidad de seguridad propietaria de este ensamblado.|
|**schema_id**|**int**| Identificador del esquema que contiene el objeto.|
|**parent_object_id**|**id**| Número de identificación del objeto primario de este flujo. En la implementación actual el valor devuelto es siempre NULL.|
|**type**|**char(2)**|Tipo de objeto. En el caso de los objetos del flujo, el tipo es siempre "EJ".|
|**type_desc**|**nvarchar(60)**| Descripción del tipo de objeto. En el caso de los objetos del flujo, el tipo es siempre "EXTERNAL_STREAMING_JOB".|
|**create_date**|**datetime**| Fecha de creación del objeto.|
|**modify_date**|**datetime**| En la implementación actual, este valor es el mismo que el de create_date para el objeto del flujo. |
|**is_ms_shipped**|**bit**| Objeto creado por un componente interno.|  
|**is_published**|**bit**| El objeto se publica.|  
|**is_schema_published**|**bit**|Solo se ha publicado el esquema del objeto.|
|**uses_ansi_nulls**|**bit**| El objeto del flujo se ha creado con la opción de base de datos SET ANSI_NULLS en ON.|
|**instrucción**|**ntext**| Texto de consulta de Stream Analytics para el trabajo de streaming. Para obtener más información, consulte [sp_create_streaming_job](overview.md). |
|**status**|**int**| Estado actual del trabajo de streaming. Los valores posibles son: <br /><br /> **Creado** = 0. El trabajo de streaming se ha creado, pero aún no se ha iniciado. <br /><br /> **Iniciando** = 1. El trabajo de streaming se encuentra en la fase de inicio. <br /><br /> **Error** = 6. Error del trabajo de streaming. Suele indicar un error grave durante el procesamiento. <br /><br /> **Detenido** = 4. El trabajo de streaming se ha detenido. <br /><br /> **Inactivo** = 7. El trabajo de streaming se está ejecutando, pero no hay ninguna entrada para procesar. <br /><br /> **Procesando** = 8. El trabajo de streaming se está ejecutando y está procesando entradas. Este estado indica un estado de mantenimiento del trabajo de streaming. <br /><br /> **Degradado** = 9. El trabajo de streaming se está ejecutando, pero se produjeron errores de serialización o deserialización de entrada o salida no graves durante el procesamiento de entrada. El trabajo de entrada seguirá ejecutándose, pero quitará las entradas que encuentren errores.|

## <a name="permissions"></a>Permisos

La visibilidad de los metadatos en las vistas de catálogo se limita a los elementos protegibles y que son propiedad de un usuario o sobre los que el usuario tiene algún permiso. Para obtener más información, consulte [Metadata Visibility Configuration](/sql/relational-databases/security/metadata-visibility-configuration/).

## <a name="see-also"></a>Consulte también

- [Streaming de T-SQL de vistas de catálogo](overview.md)
- [Vistas de catálogo (Transact-SQL)](/sql/relational-databases/system-catalog-views/catalog-views-transact-sql/)
- [Vistas del sistema (Transact-SQL)](/sql/t-sql/language-reference/)

