---
title: 'Relleno de intervalos de tiempo e imputación de los valores que faltan: Azure SQL Edge'
description: Obtenga información sobre el relleno de intervalos de tiempo y la imputación de los valores que faltan en Azure SQL Edge
keywords: SQL Edge, series temporales
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/22/2020
ms.openlocfilehash: 03896f4f7aa4e6efc78b498406e79a299318ed7a
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932051"
---
# <a name="filling-time-gaps-and-imputing-missing-values"></a>Relleno de intervalos de tiempo e imputación de los valores que faltan 

Cuando se trata con datos de series temporales, a menudo es posible que para los atributos de estos datos haya valores que faltan. También es posible que, debido a la naturaleza de los datos o a las interrupciones en la recopilación de datos, haya *intervalos* de tiempo en el conjunto de datos.

Por ejemplo, al recopilar estadísticas de utilización de energía para un Smart Device, siempre que el dispositivo no esté operativo, habrá intervalos en las estadísticas de utilización. De forma similar, en un escenario de recopilación de datos de telemetría de máquina, es posible que los diferentes sensores estén configurados para emitir datos con diferentes frecuencias, lo que da lugar a que en los sensores falten valores. Por ejemplo, si hay dos sensores, voltaje y presión, configurados con una frecuencia de 100 Hz y 10 Hz respectivamente, el sensor de voltaje emitirá los datos cada centésima de segundo, mientras que el sensor de presión solo emitirá datos cada décima de segundo.

En la tabla siguiente se describe un conjunto de elementos de telemetría de máquina, que se ha recopilado en un intervalo de un segundo. 

```
timestamp               VoltageReading  PressureReading
----------------------- --------------- ----------------
2020-09-07 06:14:41.000 164.990400      97.223600
2020-09-07 06:14:42.000 162.241300      93.992800
2020-09-07 06:14:43.000 163.271200      NULL
2020-09-07 06:14:44.000 161.368100      93.403700
2020-09-07 06:14:45.000 NULL            NULL
2020-09-07 06:14:46.000 NULL            98.364800
2020-09-07 06:14:49.000 NULL            94.098300
2020-09-07 06:14:51.000 157.695700      103.359100
2020-09-07 06:14:52.000 157.019200      NULL
2020-09-07 06:14:54.000 NULL            95.352000
2020-09-07 06:14:56.000 159.183500      100.748200

```

Hay dos características importantes en el conjunto de datos anterior. 

- El conjunto de datos no contiene ningún punto de datos relacionado con varias marcas de tiempo `2020-09-07 06:14:47.000`, `2020-09-07 06:14:48.000`, `2020-09-07 06:14:50.000`, `2020-09-07 06:14:53.000` y `2020-09-07 06:14:55.000`. Estas marcas de tiempo son *intervalos* en el conjunto de datos.  
- Faltan valores, representados como `null`, para las lecturas de voltaje y presión. 

## <a name="gap-filling"></a>Relleno de intervalos 

El relleno de intervalos es una técnica que permite crear un conjunto ordenado y contiguo de marcas de tiempo para facilitar el análisis de datos de series temporales. En Azure SQL Edge, la manera más fácil de rellenar los intervalos en el conjunto de datos de la serie temporal es definir una tabla temporal con la distribución temporal deseada y, después, realizar una operación `LEFT OUTER JOIN` o `RIGHT OUTER JOIN` en la tabla del conjunto de datos. 

Tomando los datos `MachineTelemetry` representados anteriormente como ejemplo, se puede usar la consulta siguiente a fin de generar un conjunto ordenado y contiguo de marcas de tiempo para el análisis. 

> [!NOTE]
> En la consulta siguiente se generan las filas que faltan, con los valores de marca de tiempo y los valores de `null` para los atributos. 

```sql
Create Table #SeriesGenerate(dt datetime Primary key Clustered)
GO

Declare @startdate datetime = '2020-09-07 06:14:41.000', @endtime datetime = '2020-09-07 06:14:56.000'
While (@startdate <= @endtime)
BEGIN
Insert into #SeriesGenerate values (@startdate)
set @startdate = DATEADD(SECOND, 1, @startdate)
END

Select a.dt as timestamp, b.VoltageReading, b.PressureReading 
From 
#SeriesGenerate a LEFT OUTER JOIN MachineTelemetry b 
    on a.dt = b.[timestamp]
```
En la consulta anterior se genera el resultado siguiente que contiene todas las marcas de tiempo de *un segundo* en el rango especificado.

El conjunto de resultados es el siguiente:

```

timestamp               VoltageReading    PressureReading
----------------------- ----------------- ----------------
2020-09-07 06:14:41.000 164.990400        97.223600
2020-09-07 06:14:42.000 162.241300        93.992800
2020-09-07 06:14:43.000 163.271200        NULL
2020-09-07 06:14:44.000 161.368100        93.403700
2020-09-07 06:14:45.000 NULL              NULL
2020-09-07 06:14:46.000 NULL              98.364800
2020-09-07 06:14:47.000 NULL              NULL
2020-09-07 06:14:48.000 NULL              NULL
2020-09-07 06:14:49.000 NULL              94.098300
2020-09-07 06:14:50.000 NULL              NULL
2020-09-07 06:14:51.000 157.695700        103.359100
2020-09-07 06:14:52.000 157.019200        NULL
2020-09-07 06:14:53.000 NULL              NULL
2020-09-07 06:14:54.000 NULL              95.352000
2020-09-07 06:14:55.000 NULL              NULL
2020-09-07 06:14:56.000 159.183500        100.748200
```

## <a name="imputing-missing-values"></a>Imputación de los valores que faltan

En la consulta anterior se han generado las marcas de tiempo que faltan para el análisis de datos; sin embargo, no se ha sustituido ninguno de los valores que faltan (representados como NULL) para las lecturas de `voltage` y `pressure`. En Azure SQL Edge, se ha agregado una nueva sintaxis a las funciones de T-SQL `LAST_VALUE()` y `FIRST_VALUE()`, que proporcionan mecanismos para imputar valores que faltan, en función de los valores anteriores o posteriores del conjunto de datos. 

La sintaxis nueva agrega las cláusulas `IGNORE NULLS` y `RESPECT NULLS` a las funciones `LAST_VALUE()` y `FIRST_VALUE()`. En la consulta siguiente, del conjunto de datos `MachineTelemetry`, se calculan los valores que faltan mediante la función last_value, donde los valores que faltan se reemplazan por el último valor observado en el conjunto de datos.

```sql
Select 
    timestamp,
    VoltageReading As OriginalVoltageValues,
    LAST_VALUE(VoltageReading) IGNORE NULLS OVER (ORDER BY timestamp) As ImputedUsingLastValue, 
    PressureReading As OriginalPressureValues,
    LAST_VALUE(PressureReading) IGNORE NULLS OVER (ORDER BY timestamp) As ImputedUsingLastValue
From 
MachineTelemetry 
order by timestamp 
```
El conjunto de resultados es el siguiente:

```

timestamp               OrigVoltageVals  ImputedVoltage OrigPressureVals  ImputedPressure
----------------------- ---------------- -------------- ----------------- ----------------
2020-09-07 06:14:41.000 164.990400       164.990400     97.223600         97.223600
2020-09-07 06:14:42.000 162.241300       162.241300     93.992800         93.992800
2020-09-07 06:14:43.000 163.271200       163.271200     NULL              93.992800
2020-09-07 06:14:44.000 161.368100       161.368100     93.403700         93.403700
2020-09-07 06:14:45.000 NULL             161.368100     NULL              93.403700
2020-09-07 06:14:46.000 NULL             161.368100     98.364800         98.364800
2020-09-07 06:14:49.000 NULL             161.368100     94.098300         94.098300
2020-09-07 06:14:51.000 157.695700       157.695700     103.359100        103.359100
2020-09-07 06:14:52.000 157.019200       157.019200     NULL              103.359100
2020-09-07 06:14:54.000 NULL             157.019200     95.352000         95.352000
2020-09-07 06:14:56.000 159.183500       159.183500     100.748200        100.748200

```

En la consulta siguiente se imputarán los valores que faltan mediante las funciones `LAST_VALUE()` y `FIRST_VALUE`. En el caso de la columna de salida `ImputedVoltage`, los valores que faltan se reemplazan por el último valor observado, mientras que para la columna de salida `ImputedPressure` estos se reemplazan por el valor observado siguiente del conjunto de datos. 

```sql
Select 
    dt as timestamp, 
    VoltageReading As OrigVoltageVals,
    LAST_VALUE(VoltageReading) IGNORE NULLS OVER (ORDER BY dt) As ImputedVoltage, 
    PressureReading As OrigPressureVals,
    First_VALUE(PressureReading) IGNORE NULLS OVER (ORDER BY dt ROWS 
                    BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING) As ImputedPressure
From 
(Select a.dt, b.VoltageReading,b.PressureReading  from 
    #SeriesGenerate a 
        LEFT OUTER JOIN 
    MachineTelemetry b 
        on a.dt = b.[timestamp]) A
order by timestamp
```
El conjunto de resultados es el siguiente:

```

timestamp               OrigVoltageVals  ImputedVoltage  OrigPressureVals  ImputedPressure
----------------------- ---------------- --------------- ----------------- ---------------
2020-09-07 06:14:41.000 164.990400       164.990400      97.223600         97.223600
2020-09-07 06:14:42.000 162.241300       162.241300      93.992800         93.992800
2020-09-07 06:14:43.000 163.271200       163.271200      NULL              93.403700
2020-09-07 06:14:44.000 161.368100       161.368100      93.403700         93.403700
2020-09-07 06:14:45.000 NULL             161.368100      NULL              98.364800
2020-09-07 06:14:46.000 NULL             161.368100      98.364800         98.364800
2020-09-07 06:14:47.000 NULL             161.368100      NULL              94.098300
2020-09-07 06:14:48.000 NULL             161.368100      NULL              94.098300
2020-09-07 06:14:49.000 NULL             161.368100      94.098300         94.098300
2020-09-07 06:14:50.000 NULL             161.368100      NULL              103.359100
2020-09-07 06:14:51.000 157.695700       157.695700      103.359100        103.359100
2020-09-07 06:14:52.000 157.019200       157.019200      NULL              95.352000
2020-09-07 06:14:53.000 NULL             157.019200      NULL              95.352000
2020-09-07 06:14:54.000 NULL             157.019200      95.352000         95.352000
2020-09-07 06:14:55.000 NULL             157.019200      NULL              100.748200
2020-09-07 06:14:56.000 159.183500       159.183500      100.748200        100.748200
```

> [!NOTE]
> En la consulta anterior se utiliza la función `FIRST_VALUE()` para reemplazar los valores que faltan por el valor observado siguiente. Se puede lograr el mismo resultado mediante el uso de la función `LAST_VALUE()` con una cláusula `ORDER BY <ordering_column> DESC`.

## <a name="next-steps"></a>Pasos siguientes 

- [FIRST_VALUE (Transact-SQL)](https://docs.microsoft.com/sql/t-sql/functions/first-value-transact-sql?toc=/azure/azure-sql-edge/toc.json)
- [LAST_VALUE (Transact-SQL)](https://docs.microsoft.com/sql/t-sql/functions/last-value-transact-sql?toc=/azure/azure-sql-edge/toc.json)
- [DATE_BUCKET (Transact-SQL)](date-bucket-tsql.md)
- [Funciones de agregado (Transact-SQL)](https://docs.microsoft.com/sql/t-sql/functions/aggregate-functions-transact-sql)
