---
title: Tutorial sobre la carga de datos de Azure Data Lake Storage
description: Use la instrucción COPY para cargar datos desde Azure Data Lake Storage para Synapse SQL.
services: synapse-analytics
author: kevinvngo
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 06/07/2020
ms.author: kevin
ms.reviewer: igorstan
ms.custom: azure-synapse
ms.openlocfilehash: c634b4f7ac3aa1fe83e6ab3f863e998b8d25232c
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87533944"
---
# <a name="load-data-from-azure-data-lake-storage-for-synapse-sql"></a>Carga de datos desde Azure Data Lake Storage para Synapse SQL

En esta guía se indica cómo usar el [comando COPY](https://docs.microsoft.com/sql/t-sql/statements/copy-into-transact-sql?view=azure-sqldw-latest) para cargar datos de Azure Data Lake Storage. Para obtener ejemplos rápidos sobre el uso de la instrucción COPY en todos los métodos de autenticación, consulte la siguiente documentación: [Carga de datos de forma segura mediante el uso de Synapse SQL](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/quickstart-bulk-load-copy-tsql-examples).

> [!NOTE]  
> Para proporcionar comentarios o informar de problemas sobre la instrucción COPY, envíe un mensaje de correo electrónico a la siguiente lista de distribución: sqldwcopypreview@service.microsoft.com.
>
> [!div class="checklist"]
>
> * Cree la tabla de destino para cargar datos de Azure Data Lake Storage.
> * Cree la instrucción COPY para cargar datos en el almacenamiento de datos.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="before-you-begin"></a>Antes de empezar

Antes de completar este tutorial, descargue e instale la versión más reciente de [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) (SSMS).

Para ejecutar este tutorial, necesitará:

* Un grupo de SQL. Consulte [Creación de un grupo de SQL y consulta de datos](create-data-warehouse-portal.md).
* Una cuenta de Data Lake Storage. Consulte [Introducción a Azure Data Lake Storage](../../data-lake-store/data-lake-store-get-started-portal.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json). Para esta cuenta de almacenamiento, tendrá que configurar o especificar una de las siguientes credenciales para cargar: Una clave de cuenta de almacenamiento, una clave de firma de acceso compartido (SAS), un usuario de la aplicación de Azure Directory o un usuario de AAD que tenga el rol de Azure adecuado para la cuenta de almacenamiento.

## <a name="create-the-target-table"></a>Creación de la tabla de destino

Conéctese al grupo de SQL y cree la tabla de destino en la que va a realizar la carga. En este ejemplo, vamos a crear una tabla de dimensiones de producto.

```sql
-- A: Create the target table
-- DimProduct
CREATE TABLE [dbo].[DimProduct]
(
    [ProductKey] [int] NOT NULL,
    [ProductLabel] [nvarchar](255) NULL,
    [ProductName] [nvarchar](500) NULL
)
WITH
(
    DISTRIBUTION = HASH([ProductKey]),
    CLUSTERED COLUMNSTORE INDEX
    --HEAP
);
```


## <a name="create-the-copy-statement"></a>Creación de la instrucción COPY

Conéctese al grupo de SQL y ejecute la instrucción COPY. Para obtener una lista completa de ejemplos, consulte la siguiente documentación: [Carga de datos de forma segura mediante el uso de Synapse SQL](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/quickstart-bulk-load-copy-tsql-examples).

```sql
-- B: Create and execute the COPY statement

COPY INTO [dbo].[DimProduct] 
--The column list allows you map, omit, or reorder input file columns to target table columns. 
--You can also specify the default value when there is a NULL value in the file.
--When the column list is not specified, columns will be mapped based on source and target ordinality
(
    ProductKey default -1 1,
    ProductLabel default 'myStringDefaultWhenNull' 2,
    ProductName default 'myStringDefaultWhenNull' 3
)
--The storage account location where you data is staged
FROM 'https://storageaccount.blob.core.windows.net/container/directory/'
WITH 
(
   --CREDENTIAL: Specifies the authentication method and credential access your storage account
   CREDENTIAL (IDENTITY = '', SECRET = '')
   --FILE_TYPE: Specifies the file type in your storage account location
   FILE_TYPE = 'CSV',
   --FIELD_TERMINATOR: Marks the end of each field (column) in a delimited text (CSV) file
   FIELDTERMINATOR = '|',
   --ROWTERMINATOR: Marks the end of a record in the file
   ROWTERMINATOR = '0x0A',
   --FIELDQUOTE: Specifies the delimiter for data of type string in a delimited text (CSV) file
   FIELDQUOTE = '',
   ENCODING = 'UTF8',
   DATEFORMAT = 'ymd',
   --MAXERRORS: Maximum number of reject rows allowed in the load before the COPY operation is canceled
   MAXERRORS = 10,
   --ERRORFILE: Specifies the directory where the rejected rows and the corresponding error reason should be written
   ERRORFILE = '/errorsfolder',
) OPTION (LABEL = 'COPY: ADLS tutorial');
```

## <a name="optimize-columnstore-compression"></a>Optimización de compresión de almacén de columnas

De forma predeterminada, las tablas se definen como un índice de almacén de columnas en clúster. Una vez completada una carga, puede que algunas de las filas de datos no se compriman en el almacén de columnas.  Existen varios motivos por los que esto puede ocurrir. Para aprender más, consulte el artículo sobre [administración de índices de almacén de columnas](sql-data-warehouse-tables-index.md).

Para optimizar el rendimiento de las consultas y la compresión de almacén de columnas después de una carga, vuelva a crear la tabla para obligar al índice de almacén de columnas a comprimir todas las filas.

```sql

ALTER INDEX ALL ON [dbo].[DimProduct] REBUILD;

```

## <a name="optimize-statistics"></a>Optimización de estadísticas

Es mejor crear estadísticas de columna única inmediatamente después de una carga. Hay algunas opciones para las estadísticas. Por ejemplo, si crea estadísticas de columna única en cada columna, la recompilación de todas las estadísticas puede llevar mucho tiempo. Si sabe que algunas columnas no van a estar en predicados de consulta, puede omitir la creación de estadísticas en dichas columnas.

Si decide crear estadísticas de columna única en todas las columnas de cada una de las tablas, puede usar el ejemplo de código de procedimiento almacenado `prc_sqldw_create_stats` del artículo sobre [estadísticas](sql-data-warehouse-tables-statistics.md).

El ejemplo siguiente es un buen punto de partida para la creación de estadísticas. Crea estadísticas de columna única en cada una de las columnas de la tabla de dimensiones y en cada una de las columnas de combinación de las tablas de hechos. Siempre puede agregar estadísticas de columna única o de varias columnas a otras columnas de la tabla de hechos más adelante.

## <a name="achievement-unlocked"></a>Logro conseguido.

Ha cargado correctamente datos en el almacenamiento de datos. Buen trabajo.

## <a name="next-steps"></a>Pasos siguientes
La carga de datos es el primer paso para desarrollar una solución de almacenamiento de datos mediante Azure Synapse Analytics. Consulte nuestros recursos de desarrollo.

> [!div class="nextstepaction"]
> [Obtenga información acerca de cómo desarrollar tablas para almacenar datos](sql-data-warehouse-tables-overview.md)

Para obtener más ejemplos y referencias de carga, consulte la siguiente documentación:
- [Documentación de referencia de la instrucción COPY](https://docs.microsoft.com/sql/t-sql/statements/copy-into-transact-sql?view=azure-sqldw-latest#syntax)
- [Ejemplos de la instrucción COPY para cada método de autenticación](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/quickstart-bulk-load-copy-tsql-examples)
- [Inicio rápido de la instrucción COPY para una sola tabla](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/quickstart-bulk-load-copy-tsql)
