---
title: Opciones de GROUP BY en SQL de Synapse
description: SQL de Synapse permite crear soluciones de desarrollo al implementar distintas opciones de GROUP BY.
services: synapse-analytics
author: filippopovic
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 04/15/2020
ms.author: fipopovi
ms.reviewer: jrasnick
ms.custom: ''
ms.openlocfilehash: ad2b13fbfbd9871a55efb1826fa1e978d4eeb453
ms.sourcegitcommit: 3fc3457b5a6d5773323237f6a06ccfb6955bfb2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90032306"
---
# <a name="group-by-options-in-synapse-sql"></a>Opciones de GROUP BY en SQL de Synapse
SQL de Synapse permite crear soluciones de desarrollo al implementar distintas opciones de GROUP BY. 

## <a name="what-does-group-by-do"></a>¿Qué hace GROUP BY?

La cláusula [GROUP BY](/sql/t-sql/queries/select-group-by-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) de T-SQL se usa para agregar datos a un conjunto de filas de resumen.

SQL a petición admite todo el intervalo de opciones de GROUP BY. El grupo de SQL admite un número limitado de opciones de GROUP BY.

## <a name="group-by-options-supported-in-sql-pool"></a>Opciones de GROUP BY admitidas en el grupo de SQL

GROUP BY tiene algunas opciones que el grupo de SQL no admite. Estas opciones tienen soluciones alternativas, que son las siguientes:

* GROUP BY con ROLLUP
* GROUPING SETS
* GROUP BY con CUBE

### <a name="rollup-and-grouping-sets-options"></a>Opciones Rollup y Grouping sets

Aquí, la opción más sencilla consiste en usar UNION ALL para ejecutar la acumulación en lugar de depender de la sintaxis explícita. El resultado es exactamente el mismo

En el siguiente ejemplo se usa la instrucción GROUP BY con la opción de ROLLUP:

```sql
SELECT [SalesTerritoryCountry]
,      [SalesTerritoryRegion]
,      SUM(SalesAmount)             AS TotalSalesAmount
FROM  dbo.factInternetSales s
JOIN  dbo.DimSalesTerritory t       ON s.SalesTerritoryKey       = t.SalesTerritoryKey
GROUP BY ROLLUP (
                        [SalesTerritoryCountry]
                ,       [SalesTerritoryRegion]
                )
;
```

Al utilizar ROLLUP, en el ejemplo anterior se solicitan las agregaciones siguientes:

* País y región
* Country
* Total general

Para reemplazar ROLLUP y obtener los mismos resultados, puede utilizar UNION ALL y especificar explícitamente las agregaciones necesarias:

```sql
SELECT [SalesTerritoryCountry]
,      [SalesTerritoryRegion]
,      SUM(SalesAmount) AS TotalSalesAmount
FROM  dbo.factInternetSales s
JOIN  dbo.DimSalesTerritory t     ON s.SalesTerritoryKey       = t.SalesTerritoryKey
GROUP BY
       [SalesTerritoryCountry]
,      [SalesTerritoryRegion]
UNION ALL
SELECT [SalesTerritoryCountry]
,      NULL
,      SUM(SalesAmount) AS TotalSalesAmount
FROM  dbo.factInternetSales s
JOIN  dbo.DimSalesTerritory t     ON s.SalesTerritoryKey       = t.SalesTerritoryKey
GROUP BY
       [SalesTerritoryCountry]
UNION ALL
SELECT NULL
,      NULL
,      SUM(SalesAmount) AS TotalSalesAmount
FROM  dbo.factInternetSales s
JOIN  dbo.DimSalesTerritory t     ON s.SalesTerritoryKey       = t.SalesTerritoryKey;
```

Para reemplazar GROUPING SETS, se aplica el principio de ejemplo. Basta con crear secciones UNION ALL para los niveles de agregación que desea ver.

### <a name="cube-options"></a>Opciones de Cube

Es posible crear una cláusula GROUP BY WITH CUBE con el método UNION ALL. El problema es que el código puede volverse rápidamente complicado y difícil de manejar. Para mitigar este problema, puede usar este enfoque más avanzado.

El primer paso es definir el 'cubo' que define todos los niveles de agregación que se van a crear. Tome nota de la instrucción CROSS JOIN de las dos tablas derivadas, ya que esta genera todos los niveles. El resto del código tiene la función de dar formato.

```sql
CREATE TABLE #Cube
WITH
(   DISTRIBUTION = ROUND_ROBIN
,   LOCATION = USER_DB
)
AS
WITH GrpCube AS
(SELECT    CAST(ISNULL(Country,'NULL')+','+ISNULL(Region,'NULL') AS NVARCHAR(50)) as 'Cols'
,          CAST(ISNULL(Country+',','')+ISNULL(Region,'') AS NVARCHAR(50))  as 'GroupBy'
,          ROW_NUMBER() OVER (ORDER BY Country) as 'Seq'
FROM       ( SELECT 'SalesTerritoryCountry' as Country
             UNION ALL
             SELECT NULL
           ) c
CROSS JOIN ( SELECT 'SalesTerritoryRegion' as Region
             UNION ALL
             SELECT NULL
           ) r
)
SELECT Cols
,      CASE WHEN SUBSTRING(GroupBy,LEN(GroupBy),1) = ','
            THEN SUBSTRING(GroupBy,1,LEN(GroupBy)-1)
            ELSE GroupBy
       END AS GroupBy  --Remove Trailing Comma
,Seq
FROM GrpCube;
```

La siguiente imagen muestra los resultados de [CREATE TABLE AS SELECT](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest):

![Agrupar por cubo](./media/develop-group-by-options/develop-group-by-cube.png)

El segundo paso es especificar una tabla de destino para almacenar los resultados temporales:

```sql
DECLARE
 @SQL NVARCHAR(4000)
,@Columns NVARCHAR(4000)
,@GroupBy NVARCHAR(4000)
,@i INT = 1
,@nbr INT = 0
;
CREATE TABLE #Results
(
 [SalesTerritoryCountry] NVARCHAR(50)
,[SalesTerritoryRegion]  NVARCHAR(50)
,[TotalSalesAmount]      MONEY
)
WITH
(   DISTRIBUTION = ROUND_ROBIN
,   LOCATION = USER_DB
)
;
```

El tercer paso es recorrer en bucle el cubo de columnas al realizar la agregación. La consulta se ejecutará una vez por cada fila de la tabla temporal #Cube. Los resultados se almacenan en la tabla temporal #Results:

```sql
SET @nbr =(SELECT MAX(Seq) FROM #Cube);

WHILE @i<=@nbr
BEGIN
    SET @Columns = (SELECT Cols    FROM #Cube where seq = @i);
    SET @GroupBy = (SELECT GroupBy FROM #Cube where seq = @i);

    SET @SQL ='INSERT INTO #Results
              SELECT '+@Columns+'
              ,      SUM(SalesAmount) AS TotalSalesAmount
              FROM  dbo.factInternetSales s
              JOIN  dbo.DimSalesTerritory t  
              ON s.SalesTerritoryKey = t.SalesTerritoryKey
              '+CASE WHEN @GroupBy <>''
                     THEN 'GROUP BY '+@GroupBy ELSE '' END

    EXEC sp_executesql @SQL;
    SET @i +=1;
END
```

Por último, se pueden devolver los resultados simplemente leyendo la tabla temporal #Results:

```sql
SELECT *
FROM #Results
ORDER BY 1,2,3
;
```

Al dividir el código en secciones y generar una construcción de bucle, el código resulta más fácil de administrar y mantener.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más sugerencias sobre desarrollo, vea la [información general sobre desarrollo](develop-overview.md).
