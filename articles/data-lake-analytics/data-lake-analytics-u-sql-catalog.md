---
title: Uso del catálogo de U-SQL en Azure Data Lake Analytics
description: Aprenda a usar el catálogo de U-SQL para compartir código y datos. Cree funciones con valores de tabla, vistas y tablas, y consúltelas.
services: data-lake-analytics
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.assetid: 57143396-ab86-47dd-b6f8-613ba28c28d2
ms.topic: how-to
ms.date: 05/09/2017
ms.openlocfilehash: 0ef1bff996589d09326e80b770953bd6b34966c5
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87132439"
---
# <a name="get-started-with-the-u-sql-catalog-in-azure-data-lake-analytics"></a>Introducción al catálogo de U-SQL en Azure Data Lake Analytics

## <a name="create-a-tvf"></a>Creación de una TVF

En el anterior script de U-SQL, repitió el uso de EXTRACT para leer el mismo archivo de origen. La función con valores de tabla de U-SQL (TVF) permite encapsular los datos para volver a usarlos más adelante.  

El siguiente script crea una función TVF llamada `Searchlog()` en la base de datos y el esquema predeterminados:

```usql
DROP FUNCTION IF EXISTS Searchlog;

CREATE FUNCTION Searchlog()
RETURNS @searchlog TABLE
(
            UserId          int,
            Start           DateTime,
            Region          string,
            Query           string,
            Duration        int?,
            Urls            string,
            ClickedUrls     string
)
AS BEGIN
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
RETURN;
END;
```

El siguiente script muestra cómo usar la TVF definida en el anterior script:

```usql
@res =
    SELECT
        Region,
        SUM(Duration) AS TotalDuration
    FROM Searchlog() AS S
GROUP BY Region
HAVING SUM(Duration) > 200;

OUTPUT @res
    TO "/output/SearchLog-use-tvf.csv"
    ORDER BY TotalDuration DESC
    USING Outputters.Csv();
```

## <a name="create-views"></a>Creación de vistas

Si tiene una expresión de consulta única, en lugar de una función TVF puede usar una vista de U-SQL para encapsular esa expresión.

El siguiente script crea una vista llamada `SearchlogView` en la base de datos y el esquema predeterminados:

```usql
DROP VIEW IF EXISTS SearchlogView;

CREATE VIEW SearchlogView AS  
    EXTRACT UserId          int,
            Start           DateTime,
            Region          string,
            Query           string,
            Duration        int?,
            Urls            string,
            ClickedUrls     string
    FROM "/Samples/Data/SearchLog.tsv"
USING Extractors.Tsv();
```

El siguiente script muestra el uso de la vista definida:

```usql
@res =
    SELECT
        Region,
        SUM(Duration) AS TotalDuration
    FROM SearchlogView
GROUP BY Region
HAVING SUM(Duration) > 200;

OUTPUT @res
    TO "/output/Searchlog-use-view.csv"
    ORDER BY TotalDuration DESC
    USING Outputters.Csv();
```

## <a name="create-tables"></a>Crear tablas.
De forma semejante a una tabla de base de datos relacional, U-SQL permite crear una tabla con un esquema predefinido o crear una tabla que infiere el esquema a partir de la consulta que rellena la tabla (lo que también se conoce como CREATE TABLE AS SELECT o CTAS).

Cree una base de datos y dos tablas mediante el siguiente script:

```usql
DROP DATABASE IF EXISTS SearchLogDb;
CREATE DATABASE SearchLogDb;
USE DATABASE SearchLogDb;

DROP TABLE IF EXISTS SearchLog1;
DROP TABLE IF EXISTS SearchLog2;

CREATE TABLE SearchLog1 (
            UserId          int,
            Start           DateTime,
            Region          string,
            Query           string,
            Duration        int?,
            Urls            string,
            ClickedUrls     string,

            INDEX sl_idx CLUSTERED (UserId ASC)
                DISTRIBUTED BY HASH (UserId)
);

INSERT INTO SearchLog1 SELECT * FROM master.dbo.Searchlog() AS s;

CREATE TABLE SearchLog2(
    INDEX sl_idx CLUSTERED (UserId ASC)
            DISTRIBUTED BY HASH (UserId)
) AS SELECT * FROM master.dbo.Searchlog() AS S; // You can use EXTRACT or SELECT here
```

## <a name="query-tables"></a>Consulta de tablas
Puede consultar tablas, como las creadas en el script anterior, de la misma forma que se consultan los archivos de datos. En lugar de crear un conjunto de filas mediante EXTRACT, ahora se puede hacer referencia al nombre de la tabla.

Para leer las tablas, modifique el script de transformación que usó anteriormente:

```usql
@rs1 =
    SELECT
        Region,
        SUM(Duration) AS TotalDuration
    FROM SearchLogDb.dbo.SearchLog2
GROUP BY Region;

@res =
    SELECT *
    FROM @rs1
    ORDER BY TotalDuration DESC
    FETCH 5 ROWS;

OUTPUT @res
    TO "/output/Searchlog-query-table.csv"
    ORDER BY TotalDuration DESC
    USING Outputters.Csv();
```

 >[!NOTE]
 >Actualmente, no puede ejecutar una instrucción SELECT en una tabla del mismo script que aquel donde creó la tabla.

## <a name="next-steps"></a>Pasos siguientes
* [Información general de Análisis de Microsoft Azure Data Lake](data-lake-analytics-overview.md)
* [Desarrollo de scripts U-SQL mediante Data Lake Tools for Visual Studio](data-lake-analytics-data-lake-tools-get-started.md)
* [Supervisión y solución de problemas de trabajos de Azure Data Lake Analytics con Azure Portal](data-lake-analytics-monitor-and-troubleshoot-jobs-tutorial.md)
