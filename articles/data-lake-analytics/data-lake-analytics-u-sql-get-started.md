---
title: Introducción al lenguaje U-SQL en Azure Data Lake Analytics
description: Obtenga información sobre los conceptos básicos del lenguaje U-SQL en Azure Data Lake Analytics. Escriba la primera consulta con variables para datos adicionales de archivos, transforme el conjunto de filas y agregue datos.
services: data-lake-analytics
ms.reviewer: jasonh
ms.assetid: 57143396-ab86-47dd-b6f8-613ba28c28d2
ms.service: data-lake-analytics
ms.topic: how-to
ms.date: 06/23/2017
ms.openlocfilehash: 6f2af2eed8aa01f35e1813bcd910f3ea22e2a335
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87128852"
---
# <a name="get-started-with-u-sql-in-azure-data-lake-analytics"></a>Introducción a U-SQL en Azure Data Lake Analytics

U-SQL es un lenguaje que combina SQL declarativo con C# imperativo para permitirle procesar sus datos a cualquier escala. Mediante la funcionalidad de consulta distribuida escalable de U-SQL, puede analizar los datos de manera eficaz entre almacenes relacionales, como Azure SQL Database. Con U-SQL, puede procesar los datos no estructurados mediante la aplicación de esquemas en la lectura y la inserción de lógica personalizada y UDF. Además, U-SQL incluye extensibilidad que proporciona un control más preciso sobre cómo ejecutarlo a escala.

## <a name="learning-resources"></a>Recursos de aprendizaje

* En el [Tutorial de U-SQL](https://aka.ms/usqltutorial) se proporciona un tutorial guiado de la mayor parte del lenguaje U-SQL. Se recomienda leer este documento a todos los desarrolladores que quieran aprender U-SQL.
* Para más información sobre la **sintaxis del lenguaje U-SQL**, consulte la [referencia del lenguaje U-SQL](https://docs.microsoft.com/u-sql/).
* Para comprender la **filosofía del diseño de U-SQL**, consulte la entrada del blog de Visual Studio [Introducing U-SQL – A Language that makes Big Data Processing Easy](https://blogs.msdn.microsoft.com/visualstudio/2015/09/28/introducing-u-sql-a-language-that-makes-big-data-processing-easy/) (Introducción s U-SQL: un lenguaje que facilita el procesamiento de marcrodatos).

## <a name="prerequisites"></a>Prerequisites

Antes de examinar los ejemplos de U-SQL de este documento, lea y realice el [Tutorial: Desarrollo de scripts U-SQL mediante Data Lake Tools for Visual Studio](data-lake-analytics-data-lake-tools-get-started.md). En este tutorial se explican los mecanismos de uso de U-SQL con Herramientas de Azure Data Lake para Visual Studio.

## <a name="your-first-u-sql-script"></a>El primer script U-SQL

El siguiente script U-SQL es sencillo y nos permite explorar muchos aspectos del lenguaje U-SQL.

```usql
@searchlog =
    EXTRACT UserId          int,
            Start           DateTime,
            Region          string,
            Query           string,
            Duration        int?,
            Urls            string,
            ClickedUrls     string
    FROM "/Samples/Data/SearchLog.tsv"
    USING Extractors.Tsv();

OUTPUT @searchlog
    TO "/output/SearchLog-first-u-sql.csv"
    USING Outputters.Csv();
```

Este script no contiene ningún paso de transformación. Lee el archivo de origen llamado `SearchLog.tsv`, lo esquematiza y vuelve a escribir el conjunto de filas en un archivo llamado SearchLog-first-u-sql.csv.

Observe el signo de interrogación junto al tipo de datos en el campo `Duration`. Esto significa que el campo `Duration` podría ser nulo.

### <a name="key-concepts"></a>Conceptos clave

* **Variables de conjunto de filas**: cada expresión de consulta que produce un conjunto de filas se puede asignar a una variable. U-SQL sigue el patrón de nomenclatura de variables de T-SQL (`@searchlog`, por ejemplo) en el script.
* La palabra clave **EXTRACT** lee los datos de un archivo y define el esquema en la lectura. `Extractors.Tsv` es un extractor de U-SQL integrado para archivos de valores separados por tabulación. Puede desarrollar extractores personalizados.
* **OUTPUT** escribe datos de un conjunto de filas en un archivo. `Outputters.Csv()` es un outputter U-SQL integrado para crear un archivo de valores separados por coma. También puede desarrollar outputters personalizados.

### <a name="file-paths"></a>Rutas de acceso de archivo

Las instrucciones EXTRACT y OUTPUT usan rutas de acceso de archivo. Las rutas de acceso de archivo puede ser absolutas o relativas:

La siguiente ruta de acceso absoluta de archivo hace referencia a un archivo de Data Lake Store denominado `mystore`:

```usql
adl://mystore.azuredatalakestore.net/Samples/Data/SearchLog.tsv
```

La siguiente ruta de acceso relativa de archivo comienza con `"/"`. Hace referencia a un archivo en la cuenta de Data Lake Store predeterminada:

```usql
/output/SearchLog-first-u-sql.csv
```

## <a name="use-scalar-variables"></a>Uso de variables escalares

Puede usar variables escalares para facilitar el mantenimiento del script. El script de U-SQL anterior también se puede escribir como:

```usql
DECLARE @in  string = "/Samples/Data/SearchLog.tsv";
DECLARE @out string = "/output/SearchLog-scalar-variables.csv";
@searchlog =
    EXTRACT UserId          int,
            Start           DateTime,
            Region          string,
            Query           string,
            Duration        int?,
            Urls            string,
            ClickedUrls     string
    FROM @in
    USING Extractors.Tsv();
OUTPUT @searchlog
    TO @out
    USING Outputters.Csv();
```

## <a name="transform-rowsets"></a>Transformación de conjuntos de filas

Use **SELECT** para transformar conjuntos de filas:

```usql
@searchlog =
    EXTRACT UserId          int,
            Start           DateTime,
            Region          string,
            Query           string,
            Duration        int?,
            Urls            string,
            ClickedUrls     string
    FROM "/Samples/Data/SearchLog.tsv"
    USING Extractors.Tsv();
@rs1 =
    SELECT Start, Region, Duration
    FROM @searchlog
WHERE Region == "en-gb";
OUTPUT @rs1
    TO "/output/SearchLog-transform-rowsets.csv"
    USING Outputters.Csv();
```

La cláusula WHERE usa una [expresión booleana de C#](/dotnet/csharp/language-reference/operators/index). Puede usar el lenguaje de expresiones de C# para crear sus propias expresiones y funciones. Incluso puede realizar un filtrado más complejo si las combina con conjunciones (AND) y disyunciones (OR) lógicas.

El siguiente script usa el método DateTime.Parse() y una conjunción.

```usql
@searchlog =
    EXTRACT UserId          int,
            Start           DateTime,
            Region          string,
            Query           string,
            Duration        int?,
            Urls            string,
            ClickedUrls     string
    FROM "/Samples/Data/SearchLog.tsv"
    USING Extractors.Tsv();
@rs1 =
    SELECT Start, Region, Duration
    FROM @searchlog
WHERE Region == "en-gb";
@rs1 =
    SELECT Start, Region, Duration
    FROM @rs1
    WHERE Start >= DateTime.Parse("2012/02/16") AND Start <= DateTime.Parse("2012/02/17");
OUTPUT @rs1
    TO "/output/SearchLog-transform-datetime.csv"
    USING Outputters.Csv();
```

 >[!NOTE]
 >La segunda consulta opera con el resultado del primer conjunto de filas, que crea una composición de ambos filtros. También puede volver a usar un nombre de variable y los nombres tienen un ámbito léxico.

## <a name="aggregate-rowsets"></a>Conjuntos de filas agregados

U-SQL proporciona las conocidas cláusulas ORDER BY, GROUP BY y agregaciones.

La siguiente consulta busca la duración total por región y después muestra las cinco duraciones principales de forma ordenada.

Los conjuntos de filas de U-SQL no conservan el orden en la siguiente consulta. Por lo tanto, para ordenar los resultados, necesita agregar ORDER BY a la instrucción OUTPUT:

```usql
DECLARE @outpref string = "/output/Searchlog-aggregation";
DECLARE @out1    string = @outpref+"_agg.csv";
DECLARE @out2    string = @outpref+"_top5agg.csv";
@searchlog =
    EXTRACT UserId          int,
            Start           DateTime,
            Region          string,
            Query           string,
            Duration        int?,
            Urls            string,
            ClickedUrls     string
    FROM "/Samples/Data/SearchLog.tsv"
    USING Extractors.Tsv();
@rs1 =
    SELECT
        Region,
        SUM(Duration) AS TotalDuration
    FROM @searchlog
GROUP BY Region;
@res =
    SELECT *
    FROM @rs1
    ORDER BY TotalDuration DESC
    FETCH 5 ROWS;
OUTPUT @rs1
    TO @out1
    ORDER BY TotalDuration DESC
    USING Outputters.Csv();
OUTPUT @res
    TO @out2
    ORDER BY TotalDuration DESC
    USING Outputters.Csv();
```

La cláusula ORDER BY de The U-SQL requiere el uso de la cláusula FETCH en una expresión SELECT.

La cláusula HAVING de U-SQL puede usarse para restringir los resultados a los grupos que cumplan la condición HAVING:

```usql
@searchlog =
    EXTRACT UserId          int,
            Start           DateTime,
            Region          string,
            Query           string,
            Duration        int?,
            Urls            string,
            ClickedUrls     string
    FROM "/Samples/Data/SearchLog.tsv"
    USING Extractors.Tsv();
@res =
    SELECT
        Region,
        SUM(Duration) AS TotalDuration
    FROM @searchlog
    GROUP BY Region
    HAVING SUM(Duration) > 200;
OUTPUT @res
    TO "/output/Searchlog-having.csv"
    ORDER BY TotalDuration DESC
    USING Outputters.Csv();
```

Para escenarios avanzados de agregación, consulte la documentación de referencia de U-SQL para [agregar, analizar y hacer referencia a funciones](/u-sql/built-in-functions)

## <a name="next-steps"></a>Pasos siguientes

* [Información general de Análisis de Microsoft Azure Data Lake](data-lake-analytics-overview.md)
* [Desarrollar scripts de U-SQL mediante Data Lake Tools para Visual Studio](data-lake-analytics-data-lake-tools-get-started.md)
