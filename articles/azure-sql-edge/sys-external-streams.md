---
title: 'sys.external_streams (Transact-SQL): Azure SQL Edge'
description: Información sobre el uso de sys.external_streams en Azure SQL Edge
keywords: sys.external_streams, SQL Edge
services: sql-edge
ms.service: sql-edge
ms.topic: reference
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 05/19/2019
ms.openlocfilehash: 04950f01c06bc3c8ed3bb11a790310c2319a0579
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90900303"
---
# <a name="sysexternal_streams-transact-sql"></a>sys.external_streams (Transact-SQL)

Devuelve una fila por cada objeto de flujo externo creado en el ámbito de la base de datos.

|Nombre de la columna|Tipo de datos|Descripción|  
|-----------------|---------------|-----------------|
|**name**|**sysname**|Nombre del flujo. Es único en la base de datos.|
|**object_id**|**int**|Número de identificación del objeto del flujo. Es único en la base de datos.|
|**principal_id**|**int**|Id. de la entidad de seguridad propietaria de este ensamblado.|
|**schema_id**|**int**| Identificador del esquema que contiene el objeto.|
|**parent_object_id**|**id**| Número de identificación del objeto primario de este flujo. En la implementación actual el valor devuelto es siempre NULL.|
|**type**|**char(2)**|Tipo de objeto. En el caso de los objetos de flujo, el tipo es siempre "ES".|
|**type_desc**|**nvarchar(60)**| Descripción del tipo de objeto. En el caso de los objetos de flujo, el tipo es siempre "EXTERNAL_STREAM".|
|**create_date**|**datetime**| Fecha de creación del objeto.|
|**modify_date**|**datetime**| Fecha en que se modificó el objeto por última vez con una instrucción ALTER.|
|**is_ms_shipped**|**bit**| Objeto creado por un componente interno.|  
|**is_published**|**bit**|El objeto se publica.|  
|**is_schema_published**|**bit**|Solo se ha publicado el esquema del objeto.|
|**max_column_id_used**|**bit**| Esta columna se usa para fines internos y se quitará en el futuro.|  
|**uses_ansi_nulls**|**bit**| El objeto del flujo se ha creado con la opción de base de datos SET ANSI_NULLS en ON.|
|**data_source_id**|**int**| Id. de objeto del origen de datos externo representado por el objeto de flujo. |  
|**file_format_id**|**int**| Id. de objeto del formato de archivo externo representado por el objeto del flujo. El formato de archivo externo es necesario para especificar el diseño real de los datos a los que hace referencia un flujo externo.| 
|**ubicación**|**ntext**| Destino del objeto de flujo externo. Para obtener más información, consulte [Creación del flujo externo](overview.md). |
|**input_option**|**ntext**| Opciones de entrada usadas durante la creación del flujo externo. Para obtener más información, consulte [Creación del flujo externo](overview.md). |
|**output_option**|**ntext**| Opciones de salida usadas durante la creación del flujo externo. Para obtener más información, consulte [Creación del flujo externo](overview.md). | 

## <a name="permissions"></a>Permisos

La visibilidad de los metadatos en las vistas de catálogo se limita a los elementos protegibles y que son propiedad de un usuario o sobre los que el usuario tiene algún permiso. Para obtener más información, consulte [Metadata Visibility Configuration](/sql/relational-databases/security/metadata-visibility-configuration/).

## <a name="see-also"></a>Consulte también

- [Vistas de catálogo (Transact-SQL)](/sql/relational-databases/system-catalog-views/catalog-views-transact-sql/)
- [Vistas del sistema (Transact-SQL)](/sql/t-sql/language-reference/)
