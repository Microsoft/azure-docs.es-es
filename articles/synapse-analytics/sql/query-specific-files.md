---
title: Uso de metadatos de archivo en consultas
description: La función OPENROWSET proporciona información sobre archivos y rutas de acceso de todos los archivos que se usan en la consulta para filtrar o analizar datos según el nombre de archivo o la ruta de acceso de la carpeta.
services: synapse-analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: sql
ms.date: 05/20/2020
ms.author: v-stazar
ms.reviewer: jrasnick, carlrab
ms.openlocfilehash: 04a308805dd69c4d9245d287a391f1e7fa7d50f2
ms.sourcegitcommit: 420c30c760caf5742ba2e71f18cfd7649d1ead8a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89055573"
---
# <a name="use-file-metadata-in-queries"></a>Uso de metadatos de archivo en consultas

SQL a petición puede tratar varios archivos y carpetas, tal y como se describe en el artículo [Consulta de carpetas y varios archivos](query-folders-multiple-csv-files.md). En este artículo, aprenderá a usar la información de los metadatos sobre los nombres de archivos y carpetas en las consultas.

A veces, puede que necesite saber qué origen de archivo o carpeta se correlaciona con una fila específica en el conjunto de resultados.

Puede usar las funciones `filepath` y `filename` para devolver nombres de archivo o la ruta de acceso en el conjunto de resultados. O bien puede usarlas para filtrar los datos según el nombre de archivo o la ruta de acceso de la carpeta. Estas funciones se describen en las secciones [Función filename](query-data-storage.md#filename-function) y [Función filepath](query-data-storage.md#filepath-function) de la sintaxis. A continuación encontrará breves descripciones, junto con ejemplos.

## <a name="prerequisites"></a>Prerrequisitos

El primer paso es **crear una base de datos** con un origen de datos que haga referencia a la cuenta de almacenamiento. Luego, se inicializan los objetos, para lo que hay que ejecutar un [script de instalación](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql) en esa base de datos. Este script de instalación creará los orígenes de datos, las credenciales con ámbito de base de datos y los formatos de archivo externos que se usan en estos ejemplos.

## <a name="functions"></a>Functions

### <a name="filename"></a>Nombre de archivo

Esta función devuelve el nombre de archivo del que se origina la fila.

En el ejemplo siguiente se leen los archivos de datos de NYC Yellow Taxi de los últimos tres meses de 2017 y se devuelve el número de carreras por archivo. La parte de OPENROWSET de la consulta especifica qué archivos se leerán.

```sql
SELECT
    nyc.filename() AS [filename]
    ,COUNT_BIG(*) AS [rows]
FROM  
    OPENROWSET(
        BULK 'parquet/taxi/year=2017/month=9/*.parquet',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT='PARQUET'
    ) nyc
GROUP BY nyc.filename();
```

En el ejemplo siguiente se muestra cómo se puede usar *filename()* en la cláusula WHERE para filtrar los archivos que se van a leer. Accede a toda la carpeta en la parte de OPENROWSET de la consulta y filtra los archivos en la cláusula WHERE.

Los resultados serán los mismos que en el ejemplo anterior.

```sql
SELECT
    r.filename() AS [filename]
    ,COUNT_BIG(*) AS [rows]
FROM OPENROWSET(
    BULK 'csv/taxi/yellow_tripdata_2017-*.csv',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT = 'CSV',
        PARSER_VERSION = '2.0',
        FIRSTROW = 2) 
        WITH (C1 varchar(200) ) AS [r]
WHERE
    r.filename() IN ('yellow_tripdata_2017-10.csv', 'yellow_tripdata_2017-11.csv', 'yellow_tripdata_2017-12.csv')
GROUP BY
    r.filename()
ORDER BY
    [filename];
```

### <a name="filepath"></a>Filepath

La función filepath devuelve una ruta de acceso completa o parcial:

- Cuando se la llama sin ningún parámetro, devuelve la ruta de acceso completa al archivo del que se origina la fila.
- Cuando se la llama con un parámetro, devuelve la parte de la ruta de acceso que coincide con el carácter comodín en la posición especificada del parámetro. Por ejemplo, el valor 1 del parámetro devolvería la parte de la ruta de acceso que coincide con el primer carácter comodín.

En el ejemplo siguiente se leen archivos de datos de NYC Yellow Taxi para los últimos tres meses de 2017. Devuelve el número de carreras por ruta de acceso de archivo. La parte de OPENROWSET de la consulta especifica qué archivos se leerán.

```sql
SELECT
    r.filepath() AS filepath
    ,COUNT_BIG(*) AS [rows]
FROM OPENROWSET(
        BULK 'csv/taxi/yellow_tripdata_2017-1*.csv',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT = 'CSV',
        PARSER_VERSION = '2.0',
        FIRSTROW = 2
    )
    WITH (
        vendor_id INT
    ) AS [r]
GROUP BY
    r.filepath()
ORDER BY
    filepath;
```

En el ejemplo siguiente se muestra cómo se puede usar *filepath()* en la cláusula WHERE para filtrar los archivos que se van a leer.

Puede usar los caracteres comodín en la parte de OPENROWSET de la consulta y filtrar los archivos en la cláusula WHERE. Los resultados serán los mismos que en el ejemplo anterior.

```sql
SELECT
    r.filepath() AS filepath
    ,r.filepath(1) AS [year]
    ,r.filepath(2) AS [month]
    ,COUNT_BIG(*) AS [rows]
FROM OPENROWSET(
        BULK 'csv/taxi/yellow_tripdata_*-*.csv',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT = 'CSV',
        PARSER_VERSION = '2.0',        
        FIRSTROW = 2
    )
WITH (
    vendor_id INT
) AS [r]
WHERE
    r.filepath(1) IN ('2017')
    AND r.filepath(2) IN ('10', '11', '12')
GROUP BY
    r.filepath()
    ,r.filepath(1)
    ,r.filepath(2)
ORDER BY
    filepath;
```

## <a name="next-steps"></a>Pasos siguientes

En el artículo siguiente, aprenderá a [consultar archivos de Parquet](query-parquet-files.md).
