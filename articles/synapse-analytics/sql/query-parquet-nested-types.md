---
title: Consulta de tipos anidados de Parquet con SQL a petición (versión preliminar)
description: En este artículo, aprenderá a consultar tipos anidados de Parquet con SQL a petición (versión preliminar).
services: synapse-analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: sql
ms.date: 05/20/2020
ms.author: v-stazar
ms.reviewer: jrasnick, carlrab
ms.openlocfilehash: f58adf124634ce1b4326f0026718688f0eb1dc7b
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89076742"
---
# <a name="query-nested-types-in-parquet-and-json-files-by-using-sql-on-demand-preview-in-azure-synapse-analytics"></a>Consulta de tipos anidados en Parquet y archivos JSON mediante SQL a petición (versión preliminar) en Azure Synapse Analytics

En este artículo, aprenderá a escribir una consulta mediante SQL a petición (versión preliminar) en Azure Synapse Analytics. La consulta leerá los tipos anidados de Parquet.
Los tipos anidados son estructuras complejas que representan objetos o matrices. Los tipos anidados se pueden almacenar en: 
- [Parquet](query-parquet-files.md), donde puede tener varias columnas complejas que contengan matrices y objetos.
- Los [archivos JSON](query-json-files.md) jerárquicos, donde puede leer un documento JSON complejo como una sola columna.
- Colecciones de Azure Cosmos DB (actualmente en versión preliminar pública controlada), donde todos los documentos pueden contener propiedades anidadas complejas.

Azure Synapse SQL a petición da formato a todos los tipos anidados como matrices y objetos JSON. Por lo tanto, puede [extraer o modificar objetos complejos mediante el uso de funciones JSON](https://docs.microsoft.com/sql/relational-databases/json/validate-query-and-change-json-data-with-built-in-functions-sql-server) o [analizar datos JSON mediante la función OPENJSON](https://docs.microsoft.com/sql/relational-databases/json/convert-json-data-to-rows-and-columns-with-openjson-sql-server). 

A continuación se muestra un ejemplo de una consulta que extrae valores escalares y de objetos del archivo JSON [Conjunto de datos de investigación abierto para la COVID-19](https://azure.microsoft.com/services/open-datasets/catalog/covid-19-open-research/), que contiene objetos anidados: 

```sql
SELECT
    title = JSON_VALUE(doc, '$.metadata.title'),
    first_author = JSON_QUERY(doc, '$.metadata.authors[0]'),
    first_author_name = JSON_VALUE(doc, '$.metadata.authors[0].first'),
    complex_object = doc
FROM
    OPENROWSET(
        BULK 'https://azureopendatastorage.blob.core.windows.net/covid19temp/comm_use_subset/pdf_json/000b7d1517ceebb34e1e3e817695b6de03e2fa78.json',
        FORMAT='CSV', FIELDTERMINATOR ='0x0b', FIELDQUOTE = '0x0b', ROWTERMINATOR = '0x0b'
    )
    WITH ( doc varchar(MAX) ) AS docs;
```

La función `JSON_VALUE` devuelve un valor escalar del campo en la ruta de acceso especificada. La función `JSON_QUERY` devuelve un objeto con formato JSON desde el campo en la ruta de acceso especificada.

> [!IMPORTANT]
> En este ejemplo se usa un archivo del Conjunto de datos de investigación abierto para la COVID-19. [Vea la licencia y la estructura de los datos aquí](https://azure.microsoft.com/services/open-datasets/catalog/covid-19-open-research/).

## <a name="prerequisites"></a>Requisitos previos

El primer paso es crear una base de datos en la que se creará el origen de datos. Luego, inicializará los objetos ejecutando un [script de instalación](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql) en la base de datos. El script de instalación creará los orígenes de datos, las credenciales con ámbito de base de datos y los formatos de archivo externos que se usan en los ejemplos.

## <a name="project-nested-or-repeated-data"></a>Proyección de datos anidados o repetidos de proyecto

Un archivo Parquet puede tener varias columnas con tipos complejos. Los valores de estas columnas tienen el formato de texto JSON y se devuelven como columnas VARCHAR. La columna siguiente lee el archivo structExample.parquet y muestra cómo leer los valores de las columnas anidadas: 

```sql
SELECT
    DateStruct, TimeStruct, TimestampStruct, DecimalStruct, FloatStruct
FROM
    OPENROWSET(
        BULK 'parquet/nested/structExample.parquet',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT='PARQUET'
    )
    WITH (
        DateStruct VARCHAR(8000),
        TimeStruct VARCHAR(8000),
        TimestampStruct VARCHAR(8000),
        DecimalStruct VARCHAR(8000),
        FloatStruct VARCHAR(8000)
    ) AS [r];
```

Esta consulta devuelve el siguiente resultado. El contenido de cada objeto anidado se devuelve como texto JSON.

| DateStruct    | TimeStruct    | TimestampStruct   | DecimalStruct | FloatStruct |
| --- | --- | --- | --- | --- |
|{"Date":"2009-04-25"}| {"Time":"20:51:54.3598000"}|    {"Timestamp":"5501-04-08 12:13:57.4821000"}|    {"Decimal":11143412.25350}| {"Float":0.5}|
|{"Date":"1916-04-29"}| {"Time":"00:16:04.6778000"}|    {"Timestamp":"1990-06-30 20:50:52.6828000"}|    {"Decimal":1963545.62800}|  {"Float":-2.125}|

La siguiente consulta lee el archivo justSimpleArray.parquet. Proyecta todas las columnas del archivo Parquet, incluidos los datos anidados y repetidos.

```sql
SELECT
    SimpleArray
FROM
    OPENROWSET(
        BULK 'parquet/nested/justSimpleArray.parquet',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT='PARQUET'
    ) AS [r];
```

Esta consulta devolverá el resultado siguiente:

|SimpleArray|
| --- |
|[11,12,13]|
|[21,22,23]|

## <a name="read-properties-from-nested-object-columns"></a>Lectura de propiedades desde columnas de objetos anidados

La función `JSON_VALUE` permite devolver valores desde las columnas con formato de texto JSON:

```sql
SELECT
    title = JSON_VALUE(complex_column, '$.metadata.title'),
    first_author_name = JSON_VALUE(complex_column, '$.metadata.authors[0].first'),
    body_text = JSON_VALUE(complex_column, '$.body_text.text'),
    complex_column
FROM
    OPENROWSET( BULK 'https://azureopendatastorage.blob.core.windows.net/covid19temp/comm_use_subset/pdf_json/000b7d1517ceebb34e1e3e817695b6de03e2fa78.json',
                FORMAT='CSV', FIELDTERMINATOR ='0x0b', FIELDQUOTE = '0x0b', ROWTERMINATOR = '0x0b' ) WITH ( complex_column varchar(MAX) ) AS docs;
```

El resultado se muestra en la tabla siguiente:

|title  | first_author_name | body_text | complex_column |
| --- | --- | --- | --- |
| Información complementaria Un estudio de epide… | Julien   | - Ilustración S1 : Filogenia de… | `{    "paper_id": "000b7d1517ceebb34e1e3e817695b6de03e2fa78",    "metadata": {        "title": "Supplementary Information An eco-epidemiological study of Morbilli-related paramyxovirus infection in Madagascar bats reveals host-switching as the dominant macro-evolutionary mechanism",        "authors": [            {                "first": "Julien"` |

A diferencia de los archivos JSON, que en la mayoría de los casos devuelven una sola columna que contiene un objeto JSON complejo, los archivos Parquet pueden tener varias columnas complejas. Puede leer las propiedades de las columnas anidadas mediante la función `JSON_VALUE` en cada columna. `OPENROWSET` permite especificar directamente las rutas de acceso de las propiedades anidadas en una cláusula `WITH`. Puede establecer las rutas de acceso como el nombre de una columna o puede agregar una [expresión de ruta de acceso JSON](https://docs.microsoft.com/sql/relational-databases/json/json-path-expressions-sql-server) después del tipo de columna.

La siguiente consulta lee el archivo structExample.parquet y muestra cómo exponer los elementos de una columna anidada. Hay dos formas de hacer referencia a un valor anidado:
- Especificando la expresión de ruta de acceso del valor anidado después de la especificación de tipo.
- Dando formato al nombre de la columna como una ruta de acceso anidada mediante do "." para hacer referencia a los campos.

```sql
SELECT
    *
FROM
    OPENROWSET(
        BULK 'parquet/nested/structExample.parquet',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT='PARQUET'
    )
    WITH (
        [DateValue] DATE '$.DateStruct.Date',
        [TimeStruct.Time] TIME,
        [TimestampStruct.Timestamp] DATETIME2,
        DecimalValue DECIMAL(18, 5) '$.DecimalStruct.Decimal',
        [FloatStruct.Float] FLOAT
    ) AS [r];
```

## <a name="access-elements-from-repeated-columns"></a>Acceso a elementos de columnas repetidas

La siguiente consulta lee el archivo justSimpleArray.parquet y usa [JSON_VALUE](/sql/t-sql/functions/json-value-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) para recuperar un elemento escalar de una columna repetida, como una matriz o una asignación:

```sql
SELECT
    *,
    JSON_VALUE(SimpleArray, '$[0]') AS FirstElement,
    JSON_VALUE(SimpleArray, '$[1]') AS SecondElement,
    JSON_VALUE(SimpleArray, '$[2]') AS ThirdElement
FROM
    OPENROWSET(
        BULK 'parquet/nested/justSimpleArray.parquet',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT='PARQUET'
    ) AS [r];
```

Este es el resultado:

|SimpleArray    | FirstElement  | SecondElement | ThirdElement |
| --- | --- | --- | --- |
| [11,12,13] | 11   | 12 | 13 |
| [21,22,23] | 21   | 22 | 23 |

## <a name="access-sub-objects-from-complex-columns"></a>Acceso a objetos secundarios desde columnas complejas

La consulta siguiente lee el archivo mapExample.parquet y usa [JSON_QUERY](/sql/t-sql/functions/json-query-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) para recuperar un elemento no escalar de una columna repetida, como una matriz o una asignación:

```sql
SELECT
    MapOfPersons,
    JSON_QUERY(MapOfPersons, '$."John Doe"') AS [John]
FROM
    OPENROWSET(
        BULK 'parquet/nested/mapExample.parquet',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT='PARQUET'
    ) AS [r];
```

También puede hacer referencia explícitamente a las columnas que quiere que se devuelvan en una cláusula `WITH`:

```sql
SELECT DocId,
    MapOfPersons,
    JSON_QUERY(MapOfPersons, '$."John Doe"') AS [John]
FROM
    OPENROWSET(
        BULK 'parquet/nested/mapExample.parquet',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT='PARQUET'
    ) 
    WITH (DocId bigint, MapOfPersons VARCHAR(max)) AS [r];
```

La estructura `MapOfPersons` se devuelve como una columna VARCHAR y con el formato de una cadena JSON.

## <a name="project-values-from-repeated-columns"></a>Proyección de valores de columnas repetidas

Si tiene una matriz de valores escalares (por ejemplo, `[1,2,3]`) en algunas columnas, puede expandirlos fácilmente y combinarlos con la fila principal mediante este script:

```sql
SELECT
    SimpleArray, Element
FROM
    OPENROWSET(
        BULK 'parquet/nested/justSimpleArray.parquet',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT='PARQUET'
    ) AS arrays
    CROSS APPLY OPENJSON (SimpleArray) WITH (Element int '$') as array_values
```

## <a name="next-steps"></a>Pasos siguientes

En el siguiente artículo se muestra cómo [consultar archivos JSON](query-json-files.md).
