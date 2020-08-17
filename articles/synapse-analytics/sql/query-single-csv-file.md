---
title: Consulta de archivos CSV con SQL a petición (versión preliminar)
description: En este artículo, aprenderá a consultar archivos CSV únicos con distintos formatos de archivo mediante SQL a petición (versión preliminar).
services: synapse analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: sql
ms.date: 05/20/2020
ms.author: v-stazar
ms.reviewer: jrasnick, carlrab
ms.openlocfilehash: 63755616bb524226d3c40d32b9695f4b787860d9
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87489714"
---
# <a name="query-csv-files"></a>Consulta de archivo CSV

En este artículo, aprenderá a consultar un único archivo CSV con SQL a petición (versión preliminar) en Azure Synapse Analytics. Los archivos CSV pueden tener distintos formatos: 

- Con y sin una fila de encabezado
- Valores delimitados por comas y tabulaciones
- Finales de línea al estilo de Windows y Unix
- Valores entre comillas y sin comillas, y caracteres de escape

A continuación se abordan todas las variaciones anteriores.

## <a name="quickstart-example"></a>Ejemplo de inicio rápido

La función `OPENROWSET` permite leer el contenido del archivo CSV al proporcionar la dirección URL al archivo.

### <a name="read-a-csv-file"></a>Leer un archivo CSV

La forma más fácil de ver el contenido del archivo `CSV` es proporcionar la dirección URL del archivo a la función `OPENROWSET` y especificar `FORMAT` del archivo CSV y 2.0 `PARSER_VERSION`. Si el archivo está disponible públicamente o si la identidad de Azure AD puede tener acceso a este archivo, debería poder ver el contenido del archivo mediante la consulta como la que se muestra en el ejemplo siguiente:

```sql
select top 10 *
from openrowset(
    bulk 'https://pandemicdatalake.blob.core.windows.net/public/curated/covid-19/ecdc_cases/latest/ecdc_cases.csv',
    format = 'csv',
    parser_version = '2.0',
    firstrow = 2 ) as rows
```

La opción `firstrow` se utiliza para omitir la primera fila del archivo CSV, que representa el encabezado en este caso. Asegúrese de que puede tener acceso a este archivo. Si el archivo está protegido con una clave SAS o una identidad personalizada, necesitaría configurar una [credencial de nivel de servidor para el inicio de sesión de SQL](develop-storage-files-storage-access-control.md?tabs=shared-access-signature#server-scoped-credential).

### <a name="data-source-usage"></a>Uso del origen de datos

En el ejemplo anterior se usa la ruta de acceso completa al archivo. Como alternativa, puede crear un origen de datos externo con la ubicación que apunta a la carpeta raíz del almacenamiento:

```sql
create external data source covid
with ( location = 'https://pandemicdatalake.blob.core.windows.net/public/curated/covid-19/ecdc_cases' );
```

Una vez creado el origen de datos, puede usar ese origen de datos y la ruta de acceso relativa al archivo en la función `OPENROWSET`:

```sql
select top 10 *
from openrowset(
        bulk 'latest/ecdc_cases.csv',
        data_source = 'covid',
        format = 'csv',
        parser_version ='2.0',
        firstrow = 2
    ) as rows
```

Si un origen de datos está protegido con una clave SAS o una identidad personalizada, puede configurar el [origen de datos con una credencial de ámbito de base de datos](develop-storage-files-storage-access-control.md?tabs=shared-access-signature#database-scoped-credential).

### <a name="explicitly-specify-schema"></a>Especificación explícita del esquema

`OPENROWSET` permite especificar explícitamente qué columnas desea leer del archivo con la cláusula `WITH`:

```sql
select top 10 *
from openrowset(
        bulk 'latest/ecdc_cases.csv',
        data_source = 'covid',
        format = 'csv',
        parser_version ='2.0',
        firstrow = 2
    ) with (
        date_rep date 1,
        cases int 5,
        geo_id varchar(6) 8
    ) as rows
```

Los números posteriores a un tipo de datos de la cláusula `WITH` representan el índice de la columna en el archivo CSV.

En las secciones siguientes, puede ver cómo consultar varios tipos de archivos CSV.

## <a name="prerequisites"></a>Requisitos previos

El primer paso es **crear la base de datos** en la que se crearán las tablas. Luego, se inicializan los objetos, para lo que hay que ejecutar un [script de instalación](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql) en esa base de datos. Este script de instalación creará los orígenes de datos, las credenciales con ámbito de base de datos y los formatos de archivo externos que se usan en estos ejemplos.

## <a name="windows-style-new-line"></a>Nueva línea al estilo de Windows

La consulta siguiente muestra cómo leer un archivo CSV sin una fila de encabezado, con una nueva línea al estilo de Windows y columnas delimitadas por comas.

Vista previa del archivo:

![Primeras 10 filas del archivo CSV sin encabezado, nueva línea al estilo de Windows.](./media/query-single-csv-file/population.png)

```sql
SELECT *
FROM OPENROWSET(
        BULK 'csv/population/population.csv',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT = 'CSV', PARSER_VERSION = '2.0',
        FIELDTERMINATOR =',',
        ROWTERMINATOR = '\n'
    )
WITH (
    [country_code] VARCHAR (5) COLLATE Latin1_General_BIN2,
    [country_name] VARCHAR (100) COLLATE Latin1_General_BIN2,
    [year] smallint,
    [population] bigint
) AS [r]
WHERE
    country_name = 'Luxembourg'
    AND year = 2017;
```

## <a name="unix-style-new-line"></a>Nueva línea al estilo de Unix

La consulta siguiente muestra cómo leer un archivo sin una fila de encabezado, con una nueva línea al estilo de Unix y columnas delimitadas por comas. Tenga en cuenta la ubicación diferente del archivo en comparación con los demás ejemplos.

Vista previa del archivo:

![Primeras 10 filas del archivo CSV sin fila de encabezado y con nueva línea al estilo de Unix.](./media/query-single-csv-file/population-unix.png)

```sql
SELECT *
FROM OPENROWSET(
        BULK 'csv/population-unix/population.csv',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT = 'CSV', PARSER_VERSION = '2.0',
        FIELDTERMINATOR =',',
        ROWTERMINATOR = '0x0a'
    )
WITH (
    [country_code] VARCHAR (5) COLLATE Latin1_General_BIN2,
    [country_name] VARCHAR (100) COLLATE Latin1_General_BIN2,
    [year] smallint,
    [population] bigint
) AS [r]
WHERE
    country_name = 'Luxembourg'
    AND year = 2017;
```

## <a name="header-row"></a>Fila de encabezado

La consulta siguiente muestra cómo leer un archivo con una fila de encabezado, con una nueva línea al estilo de Unix y columnas delimitadas por comas. Tenga en cuenta la ubicación diferente del archivo en comparación con los demás ejemplos.

Vista previa del archivo:

![Primeras 10 filas del archivo CSV con una fila de encabezado y con nueva línea al estilo de Unix.](./media/query-single-csv-file/population-unix-hdr.png)

```sql
SELECT *
FROM OPENROWSET(
        BULK 'csv/population-unix-hdr/population.csv',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT = 'CSV', PARSER_VERSION = '2.0',
        FIELDTERMINATOR =',',
        FIRSTROW = 2
    )
    WITH (
        [country_code] VARCHAR (5) COLLATE Latin1_General_BIN2,
        [country_name] VARCHAR (100) COLLATE Latin1_General_BIN2,
        [year] smallint,
        [population] bigint
    ) AS [r]
WHERE
    country_name = 'Luxembourg'
    AND year = 2017;
```

## <a name="custom-quote-character"></a>Carácter de comillas personalizado

La consulta siguiente muestra cómo leer un archivo con una fila de encabezado, con una nueva línea al estilo de Unix, columnas delimitadas por comas y valores entre comillas. Tenga en cuenta la ubicación diferente del archivo en comparación con los demás ejemplos.

Vista previa del archivo:

![Primeras 10 filas del archivo CSV con una fila de encabezado, con nueva línea al estilo de Unix y valores entre comillas.](./media/query-single-csv-file/population-unix-hdr-quoted.png)

```sql
SELECT *
FROM OPENROWSET(
        BULK 'csv/population-unix-hdr-quoted/population.csv',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT = 'CSV', PARSER_VERSION = '2.0',
        FIELDTERMINATOR =',',
        ROWTERMINATOR = '0x0a',
        FIRSTROW = 2,
        FIELDQUOTE = '"'
    )
    WITH (
        [country_code] VARCHAR (5) COLLATE Latin1_General_BIN2,
        [country_name] VARCHAR (100) COLLATE Latin1_General_BIN2,
        [year] smallint,
        [population] bigint
    ) AS [r]
WHERE
    country_name = 'Luxembourg'
    AND year = 2017;
```

> [!NOTE]
> Esta consulta devolvería los mismos resultados si se omitiese el parámetro FIELDQUOTE, ya que el valor predeterminado de FIELDQUOTE es una comilla doble.

## <a name="escape-characters"></a>Carácter de escape

La consulta siguiente muestra cómo leer un archivo con una fila de encabezado, con una nueva línea al estilo de Unix, columnas delimitadas por comas y un carácter de escape usado para el delimitador de campos (coma) sin valores. Tenga en cuenta la ubicación diferente del archivo en comparación con los demás ejemplos.

Vista previa del archivo:

![Primeras 10 filas del archivo CSV con una fila de encabezado, con nueva línea al estilo de Unix y un carácter de escape usado para el delimitador de campos.](./media/query-single-csv-file/population-unix-hdr-escape.png)

```sql
SELECT *
FROM OPENROWSET(
        BULK 'csv/population-unix-hdr-escape/population.csv',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT = 'CSV', PARSER_VERSION = '2.0',
        FIELDTERMINATOR =',',
        ROWTERMINATOR = '0x0a',
        FIRSTROW = 2,
        ESCAPECHAR = '\\'
    )
    WITH (
        [country_code] VARCHAR (5) COLLATE Latin1_General_BIN2,
        [country_name] VARCHAR (100) COLLATE Latin1_General_BIN2,
        [year] smallint,
        [population] bigint
    ) AS [r]
WHERE
    country_name = 'Slovenia';
```

> [!NOTE]
> Esta consulta produciría un error si no se especificara ESCAPECHAR, ya que la coma en "Slov,enia" se trataría como delimitador de campos en lugar de como parte del nombre del país o región. "Slov,enia" se trataría como dos columnas. Por lo tanto, la fila en particular tendría una columna más que las demás filas y una columna más de la definida en la cláusula WITH.

### <a name="escape-quoting-characters"></a>Caracteres de comillas de escape

La consulta siguiente muestra cómo leer un archivo con una fila de encabezado, con una nueva línea al estilo de Unix, columnas delimitadas por comas y un carácter de comillas dobles de escape entre valores. Tenga en cuenta la ubicación diferente del archivo en comparación con los demás ejemplos.

Vista previa del archivo:

![La consulta siguiente muestra cómo leer un archivo con una fila de encabezado, con una nueva línea al estilo de Unix, columnas delimitadas por comas y un carácter de comillas dobles de escape entre valores.](./media/query-single-csv-file/population-unix-hdr-escape-quoted.png)

```sql
SELECT *
FROM OPENROWSET(
        BULK 'csv/population-unix-hdr-escape-quoted/population.csv',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT = 'CSV', PARSER_VERSION = '2.0',
        FIELDTERMINATOR =',',
        ROWTERMINATOR = '0x0a',
        FIRSTROW = 2
    )
    WITH (
        [country_code] VARCHAR (5) COLLATE Latin1_General_BIN2,
        [country_name] VARCHAR (100) COLLATE Latin1_General_BIN2,
        [year] smallint,
        [population] bigint
    ) AS [r]
WHERE
    country_name = 'Slovenia';
```

> [!NOTE]
> El carácter de comillas se debe escapar con otro carácter de comillas. El carácter de comillas solo puede aparecer dentro del valor de la columna si el valor se encapsula entre caracteres de comillas.

## <a name="tab-delimited-files"></a>Archivos delimitados por tabulaciones

La consulta siguiente muestra cómo leer un archivo con una fila de encabezado, con una nueva línea al estilo de Unix y columnas delimitadas por tabulaciones. Tenga en cuenta la ubicación diferente del archivo en comparación con los demás ejemplos.

Vista previa del archivo:

![Primeras 10 filas del archivo CSV con una fila de encabezado, con nueva línea al estilo de Unix y delimitador de tabulaciones.](./media/query-single-csv-file/population-unix-hdr-tsv.png)

```sql
SELECT *
FROM OPENROWSET(
        BULK 'csv/population-unix-hdr-tsv/population.csv',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT = 'CSV', PARSER_VERSION = '2.0',
        FIELDTERMINATOR ='\t',
        ROWTERMINATOR = '0x0a',
        FIRSTROW = 2
    )
    WITH (
        [country_code] VARCHAR (5) COLLATE Latin1_General_BIN2,
        [country_name] VARCHAR (100) COLLATE Latin1_General_BIN2,
        [year] smallint,
        [population] bigint
    ) AS [r]
WHERE
    country_name = 'Luxembourg'
    AND year = 2017
```

## <a name="return-a-subset-of-columns"></a>Devolver un subconjunto de columnas

Hasta ahora, ha especificado el esquema de archivos CSV mediante WITH y enumerando todas las columnas. Solo puede especificar las columnas que realmente necesita en la consulta mediante un número ordinal para cada columna necesaria. También omitirá las columnas que no sean de interés.

La consulta siguiente devuelve el número de nombres de país distintos incluidos en un archivo, especificando solo las columnas necesarias:

> [!NOTE]
> Eche un vistazo a la cláusula WITH en la consulta siguiente y observe que hay "2" (sin comillas) al final de la fila en la que se define la columna *[country_name]* . Significa que la columna *[country_name]* es la segunda columna del archivo. La consulta omitirá todas las columnas del archivo excepto la segunda.

```sql
SELECT
    COUNT(DISTINCT country_name) AS countries
FROM OPENROWSET(
        BULK 'csv/population/population.csv',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT = 'CSV', PARSER_VERSION = '2.0',
        FIELDTERMINATOR =',',
        ROWTERMINATOR = '\n'
    )
WITH (
    --[country_code] VARCHAR (5),
    [country_name] VARCHAR (100) 2
    --[year] smallint,
    --[population] bigint
) AS [r]
```

## <a name="next-steps"></a>Pasos siguientes

En los artículos siguientes obtendrá información sobre:

- [Consulta de archivos de Parquet](query-parquet-files.md)
- [Consulta de carpetas y varios archivos](query-folders-multiple-csv-files.md)
