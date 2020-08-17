---
title: Consulta de archivos de Parquet con SQL a petición (versión preliminar)
description: En este artículo, aprenderá a consultar archivos de Parquet con SQL a petición (versión preliminar).
services: synapse analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: sql
ms.date: 05/20/2020
ms.author: v-stazar
ms.reviewer: jrasnick, carlrab
ms.openlocfilehash: 8083edaf647f52a07d55dddf21fe5751340783be
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87496243"
---
# <a name="query-parquet-files-using-sql-on-demand-preview-in-azure-synapse-analytics"></a>Consulta de archivos de Parquet con SQL a petición (versión preliminar) en Azure Synapse Analytics

En este artículo, aprenderá a escribir una consulta con SQL a petición (versión preliminar) que leerá archivos de Parquet.

## <a name="quickstart-example"></a>Ejemplo de inicio rápido

La función `OPENROWSET` permite leer el contenido del archivo Parquet al proporcionar la dirección URL al archivo.

### <a name="read-parquet-file"></a>Lectura del archivo de Parquet

La forma más fácil de ver el contenido del archivo `PARQUET` es proporcionar la dirección URL del archivo a la función `OPENROWSET` y especificar el `FORMAT` de Parquet. Si el archivo está disponible públicamente o si la identidad de Azure AD puede tener acceso a este archivo, debería poder ver el contenido del archivo mediante la consulta como la que se muestra en el ejemplo siguiente:

```sql
select top 10 *
from openrowset(
    bulk 'https://pandemicdatalake.blob.core.windows.net/public/curated/covid-19/ecdc_cases/latest/ecdc_cases.parquet',
    format = 'parquet') as rows
```

Asegúrese de que tiene acceso a este archivo. Si el archivo está protegido con una clave SAS o una identidad personalizada de Azure, necesitaría configurar una [credencial de nivel de servidor para el inicio de sesión de SQL](develop-storage-files-storage-access-control.md?tabs=shared-access-signature#server-scoped-credential).

### <a name="data-source-usage"></a>Uso del origen de datos

En el ejemplo anterior se usa la ruta de acceso completa al archivo. Como alternativa, puede crear un origen de datos externo con la ubicación que apunta a la carpeta raíz del almacenamiento y usar ese origen de datos y la ruta de acceso relativa al archivo en la función `OPENROWSET`:

```sql
create external data source covid
with ( location = 'https://pandemicdatalake.blob.core.windows.net/public/curated/covid-19/ecdc_cases' );
go
select top 10 *
from openrowset(
        bulk 'latest/ecdc_cases.parquet',
        data_source = 'covid',
        format = 'parquet'
    ) as rows
```

Si un origen de datos está protegido con una clave SAS o una identidad personalizada, puede configurar el [origen de datos con una credencial de ámbito de base de datos](develop-storage-files-storage-access-control.md?tabs=shared-access-signature#database-scoped-credential).

### <a name="explicitly-specify-schema"></a>Especificación explícita del esquema

`OPENROWSET` permite especificar explícitamente qué columnas desea leer del archivo con la cláusula `WITH`:

```sql
select top 10 *
from openrowset(
        bulk 'latest/ecdc_cases.parquet',
        data_source = 'covid',
        format = 'parquet'
    ) with ( date_rep date, cases int, geo_id varchar(6) ) as rows
```

En las secciones siguientes, puede ver cómo consultar varios tipos de archivos PARQUET.

## <a name="prerequisites"></a>Prerrequisitos

El primer paso consiste en **crear una base de datos** con un origen de datos que haga referencia a la cuenta de almacenamiento de [NYC Yellow Taxi](https://azure.microsoft.com/services/open-datasets/catalog/nyc-taxi-limousine-commission-yellow-taxi-trip-records/). Luego, se inicializan los objetos, para lo que hay que ejecutar un [script de instalación](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql) en esa base de datos. Este script de instalación creará los orígenes de datos, las credenciales con ámbito de base de datos y los formatos de archivo externos que se usan en estos ejemplos.

## <a name="dataset"></a>Dataset

En este ejemplo se usa el conjunto de datos [NYC Yellow Taxi](https://azure.microsoft.com/services/open-datasets/catalog/nyc-taxi-limousine-commission-yellow-taxi-trip-records/). Puede consultar los archivos Parquet de la misma manera en que [lee archivos CSV](query-parquet-files.md). La única diferencia es que el parámetro `FILEFORMAT` debe establecerse en `PARQUET`. En los ejemplos de este artículo se muestran los detalles de la lectura de archivos de Parquet.

## <a name="query-set-of-parquet-files"></a>Consulta de conjunto de archivos de Parquet

Solo puede especificar las columnas de interés al consultar los archivos de Parquet.

```sql
SELECT
        YEAR(tpepPickupDateTime),
        passengerCount,
        COUNT(*) AS cnt
FROM  
    OPENROWSET(
        BULK 'puYear=2018/puMonth=*/*.snappy.parquet',
        DATA_SOURCE = 'YellowTaxi',
        FORMAT='PARQUET'
    ) WITH (
        tpepPickupDateTime DATETIME2,
        passengerCount INT
    ) AS nyc
GROUP BY
    passengerCount,
    YEAR(tpepPickupDateTime)
ORDER BY
    YEAR(tpepPickupDateTime),
    passengerCount;
```

## <a name="automatic-schema-inference"></a>Inferencia automática del esquema

No es necesario usar la cláusula OPENROWSET WITH al leer los archivos de Parquet. Los nombres de columna y los tipos de datos se leen automáticamente desde los archivos de Parquet.

En el ejemplo siguiente se muestran las funcionalidades de inferencia automática del esquema para los archivos de Parquet. Devuelve el número de filas de septiembre de 2017 sin especificar un esquema.

> [!NOTE]
> No es necesario especificar columnas en la cláusula OPENROWSET WITH al leer los archivos de Parquet. En dicho caso, el servicio de consulta de SQL a petición utilizará los metadatos en el archivo de Parquet y enlazará las columnas por nombre.

```sql
SELECT TOP 10 *
FROM  
    OPENROWSET(
        BULK 'puYear=2018/puMonth=*/*.snappy.parquet',
        DATA_SOURCE = 'YellowTaxi',
        FORMAT='PARQUET'
    ) AS nyc
```

### <a name="query-partitioned-data"></a>Consulta de datos con particiones

El conjunto de datos que se proporciona en este ejemplo se divide (particiona) en subcarpetas independientes. Puede dirigirse a particiones específicas mediante la función filepath. Este ejemplo muestra los importes de las tarifas por year, month y payment_type durante los tres primeros meses de 2017.

> [!NOTE]
> La consulta de SQL a petición es compatible con el esquema de partición de Hive y Hadoop.

```sql
SELECT
        YEAR(tpepPickupDateTime),
        passengerCount,
        COUNT(*) AS cnt
FROM  
    OPENROWSET(
        BULK 'puYear=*/puMonth=*/*.snappy.parquet',
        DATA_SOURCE = 'YellowTaxi',
        FORMAT='PARQUET'
    ) nyc
WHERE
    nyc.filepath(1) = 2017
    AND nyc.filepath(2) IN (1, 2, 3)
    AND tpepPickupDateTime BETWEEN CAST('1/1/2017' AS datetime) AND CAST('3/31/2017' AS datetime)
GROUP BY
    passengerCount,
    YEAR(tpepPickupDateTime)
ORDER BY
    YEAR(tpepPickupDateTime),
    passengerCount;
```

## <a name="type-mapping"></a>Asignación de tipos

Los archivos de Parquet contienen descripciones de tipos para cada columna. En la tabla siguiente se describe cómo se asignan los tipos de Parquet a los tipos nativos de SQL.

| Tipo de Parquet | Tipo lógico de Parquet (anotación) | Tipo de datos de SQL |
| --- | --- | --- |
| BOOLEAN | | bit |
| BINARY/BYTE_ARRAY | | varbinary |
| DOUBLE | | FLOAT |
| FLOAT | | real |
| INT32 | | int |
| INT64 | | bigint |
| INT96 | |datetime2 |
| FIXED_LEN_BYTE_ARRAY | |binary |
| BINARY |UTF8 |varchar \*(intercalación UTF8) |
| BINARY |STRING |varchar \*(intercalación UTF8) |
| BINARY |ENUM|varchar \*(intercalación UTF8) |
| BINARY |UUID |UNIQUEIDENTIFIER |
| BINARY |DECIMAL |Decimal |
| BINARY |JSON |varchar(max) \*(intercalación UTF8) |
| BINARY |BSON |varbinary(max) |
| FIXED_LEN_BYTE_ARRAY |DECIMAL |Decimal |
| BYTE_ARRAY |INTERVAL |VARCHAR(max), serializado en formato normalizado |
| INT32 |INT(8, true) |SMALLINT |
| INT32 |INT(16, true) |SMALLINT |
| INT32 |INT(32, true) |int |
| INT32 |INT(8, false) |TINYINT |
| INT32 |INT(16, false) |int |
| INT32 |INT(32, false) |bigint |
| INT32 |DATE |date |
| INT32 |DECIMAL |Decimal |
| INT32 |TIME (MILLIS)|time |
| INT64 |INT(64, true) |bigint |
| INT64 |INT(64, false) |decimal(20,0) |
| INT64 |DECIMAL |Decimal |
| INT64 |TIME (MICROS / NANOS) |time |
|INT64 |TIMESTAMP (MILLIS / MICROS / NANOS) |datetime2 |
|[Tipo complejo](https://github.com/apache/parquet-format/blob/master/LogicalTypes.md#lists) |LISTA |varchar(max), serializado en JSON |
|[Tipo complejo](https://github.com/apache/parquet-format/blob/master/LogicalTypes.md#maps)|MAP|varchar(max), serializado en JSON |

## <a name="next-steps"></a>Pasos siguientes

Pase al siguiente artículo para obtener más información sobre cómo [consultar los tipos anidados de Parquet](query-parquet-nested-types.md).
