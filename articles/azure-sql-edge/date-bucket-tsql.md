---
title: 'Date_Bucket (Transact-SQL): Azure SQL Edge'
description: Más información sobre el uso de Date_Bucket en Azure SQL Edge
keywords: Date_Bucket, SQL Edge
services: sql-edge
ms.service: sql-edge
ms.topic: reference
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/03/2020
ms.openlocfilehash: 896caae2dfd79c4678ffb34c531fb56835e9bd66
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90886844"
---
# <a name="date_bucket-transact-sql"></a>Date_Bucket (Transact-SQL)

Esta función devuelve el valor de fecha y hora correspondiente al inicio de cada cubo de fecha y hora, desde la marca de tiempo definida por el parámetro `origin` o el valor de origen predeterminado de `1900-01-01 00:00:00.000`, si el parámetro de origen no se ha especificado. 

Para una introducción acerca de todos los tipos de datos y funciones de fecha y hora de Transact-SQL, vea [Tipos de datos y funciones de fecha y hora &#40;Transact-SQL&#41;](/sql/t-sql/functions/date-and-time-data-types-and-functions-transact-sql/).

[Convenciones de sintaxis de Transact-SQL](/sql/t-sql/language-elements/transact-sql-syntax-conventions-transact-sql/)

## <a name="syntax"></a>Sintaxis

```sql
DATE_BUCKET (datePart, number, date, origin)
```

## <a name="arguments"></a>Argumentos

*datePart*

Parte de *date* que se usa con el parámetro ‘number’. Por ejemplo, Año, mes, minuto, segundo, etc.

> [!NOTE]
> `DATE_BUCKET` no acepta los equivalentes de variables definidas por el usuario para los argumentos *datePart*.
  
|*datePart*|Abreviaturas|  
|---|---|
|**day**|**dd**, **d**|  
|**week**|**wk**, **ww**|  
|**hour**|**hh**|  
|**minute**|**mi**, **n**|  
|**second**|**ss**, **s**|  
|**millisecond**|**ms**|  

*número*

Número entero que decide el ancho del cubo combinado con el argumento *datePart*. Representa el ancho de los cubos de datePart desde la hora de origen. **`This argument cannot be a negative integer value`**. 

*date*

Una expresión que se puede resolver en uno de los valores siguientes:

+ **date**
+ **datetime**
+ **datetimeoffset**
+ **datetime2**
+ **smalldatetime**
+ **time**

Para *data*, `DATE_BUCKET` aceptará una expresión de columna, una expresión o una variable definida por el usuario si se resuelven en cualquiera de los tipos de datos mencionados anteriormente.

**Origen** 

Expresión opcional que se puede resolver en uno de los valores siguientes:

+ **date**
+ **datetime**
+ **datetimeoffset**
+ **datetime2**
+ **smalldatetime**
+ **time**

El tipo de datos de `Origin` debe coincidir con el tipo de datos del parámetro `Date`. 

`DATE_BUCKET` usa un valor de fecha de origen predeterminado de `1900-01-01 00:00:00.000`, es decir, 12:00 a.m. del lunes, 1 de enero de 1900, si no se especifica ningún valor de origen para la función.

## <a name="return-type"></a>Tipo de valor devuelto

El tipo de datos de valores devueltos de este método es dinámico. El tipo devuelto depende del argumento proporcionado para `date`. Si se proporciona otro tipo válido de datos de entrada para `date`, `DATE_BUCKET` devuelve el mismo tipo de datos. `DATE_BUCKET` genera un error si se especifica un literal de cadena para el parámetro `date`.

## <a name="return-values"></a>Valores devueltos

### <a name="understanding-the-output-from-date_bucket"></a>Descripción del resultado de `DATE_BUCKET`

`Date_Bucket` devuelve el valor de fecha u hora más reciente, que corresponde a datePart y al parámetro number. Por ejemplo, en las expresiones siguientes, `Date_Bucket` devolverá el valor de salida de `2020-04-13 00:00:00.0000000`, ya que la salida se calcula en función de los cubos de una semana a partir de la hora de origen predeterminada de `1900-01-01 00:00:00.000`. El valor `2020-04-13 00:00:00.0000000` es 6276 semanas desde el valor de origen de `1900-01-01 00:00:00.000`. 

```sql
declare @date datetime2 = '2020-04-15 21:22:11'
Select DATE_BUCKET(wk, 1, @date)
```

Para todas las expresiones siguientes, se devolverá el mismo valor de salida de `2020-04-13 00:00:00.0000000`. Esto se debe a que `2020-04-13 00:00:00.0000000` es 6276 semanas a partir de la fecha de origen y 6276 es divisible por 2, 3, 4 y 6.

```sql
declare @date datetime2 = '2020-04-15 21:22:11'
Select DATE_BUCKET(wk, 2, @date)
Select DATE_BUCKET(wk, 3, @date)
Select DATE_BUCKET(wk, 4, @date)
Select DATE_BUCKET(wk, 6, @date)
```

La salida de la expresión siguiente es `2020-04-06 00:00:00.0000000`, que es 6275 semanas a partir de la hora de origen predeterminada `1900-01-01 00:00:00.000`.

```sql
declare @date datetime2 = '2020-04-15 21:22:11'
Select DATE_BUCKET(wk, 5, @date)
```

La salida de la expresión siguiente es `2020-06-09 00:00:00.0000000`, que es 75 semanas a partir de la hora de origen especificada `2019-01-01 00:00:00`.

```sql
declare @date datetime2 = '2020-06-15 21:22:11'
declare @origin datetime2 = '2019-01-01 00:00:00'
Select DATE_BUCKET(wk, 5, @date, @origin)
```

## <a name="datepart-argument"></a>Argumento datepart

**dayofyear**, **day** y **weekday** devuelven el mismo valor. Cada *datepart* y sus abreviaturas devuelven el mismo valor.
  
## <a name="number-argument"></a>Argumento number

El argumento *number* no puede superar el intervalo de valores **int** positivos. En estas instrucciones, el argumento para *number* supera el intervalo de **int** en uno. La siguiente instrucción devuelve el mensaje de error siguiente: "`Msg 8115, Level 16, State 2, Line 2. Arithmetic overflow error converting expression to data type int."`
  
```sql
declare @date datetime2 = '2020-04-30 00:00:00'
Select DATE_BUCKET(dd, 2147483648, @date)
```  

Si se pasa un valor negativo para number a la función `Date_Bucket`, se devolverá el siguiente error. 

```sql
Msg 9834, Level 16, State 1, Line 1
Invalid bucket width value passed to date_bucket function. Only positive values are allowed.
````

## <a name="date-argument"></a>Argumento date  

`DATE_BUCKET` devuelve el valor base correspondiente al tipo de datos del argumento `date`. En el ejemplo siguiente, se devuelve un valor de salida con el tipo de datos datetime2. 

```sql
Select DATE_BUCKET(dd, 10, SYSUTCDATETIME())
```

## <a name="origin-argument"></a>Argumento origin  

El tipo de datos de los argumentos `origin` y `date` debe ser el mismo. Si se usan tipos de datos diferentes, se generará un error.

## <a name="remarks"></a>Observaciones

Use `DATE_BUCKET` en las cláusulas siguientes:

+ GROUP BY
+ HAVING
+ ORDER BY
+ SELECT \<list>
+ WHERE

## <a name="examples"></a>Ejemplos

### <a name="a-calculating-date_bucket-with-a-bucket-width-of-1-from-the-origin-time"></a>A. Cálculo de Date_Bucket con un ancho de cubo de 1 a partir de la hora de origen

Cada una de estas instrucciones incrementa *date_bucket* con un ancho de cubo de 1 a partir de la hora de origen:

```sql
declare @date datetime2 = '2020-04-30 21:21:21'
Select 'Week',  DATE_BUCKET(wk, 1, @date)
Union All
Select 'Day',  DATE_BUCKET(dd, 1, @date)
Union All
Select 'Hour',  DATE_BUCKET(hh, 1, @date)
Union All
Select 'Minutes',  DATE_BUCKET(mi, 1, @date)
Union All
Select 'Seconds',  DATE_BUCKET(ss, 1, @date)
```

El conjunto de resultados es el siguiente:

```sql
Week    2020-04-27 00:00:00.0000000
Day     2020-04-30 00:00:00.0000000
Hour    2020-04-30 21:00:00.0000000
Minutes 2020-04-30 21:21:00.0000000
Seconds 2020-04-30 21:21:21.0000000
```

### <a name="b-using-expressions-as-arguments-for-the-number-and-date-parameters"></a>B. Utilizar las expresiones como argumentos para los parámetros number y date

En los ejemplos siguientes se usan otros tipos de expresiones como argumentos para los parámetros *number* y *date*. Estos ejemplos se compilan con la base de datos "AdventureWorksDW2017".
  
#### <a name="specifying-user-defined-variables-as-number-and-date"></a>Especificar las variables definidas por el usuario como number y date  

En este ejemplo se especifican variables definidas por el usuario como argumentos para *number* y *date*:
  
```sql
DECLARE @days int = 365,
        @datetime datetime2 = '2000-01-01 01:01:01.1110000'; /* 2000 was a leap year */;  
SELECT Date_Bucket(day, @days, @datetime);
```

El conjunto de resultados es el siguiente:

```sql
---------------------------
1999-12-08 00:00:00.0000000

(1 row affected)
```  

#### <a name="specifying-a-column-as-date"></a>Especificar una columna como fecha

En el ejemplo siguiente, se calcula la suma de OrderQuantity y la suma de UnitPrice agrupadas en cubos de fechas semanales.
  
```sql
SELECT
    Date_Bucket(week, 1 ,cast(Shipdate as datetime2)) AS ShippedDateBucket
    ,Sum(OrderQuantity)  As SumOrderQuantity
    ,Sum(UnitPrice) As SumUnitPrice
FROM dbo.FactInternetSales FIS
where Shipdate between '2011-01-03 00:00:00.000' and '2011-02-28 00:00:00.000'
Group by Date_Bucket(week, 1 ,cast(Shipdate as datetime2))
order by 1
```  

El conjunto de resultados es el siguiente:
  
```sql
ShippedDateBucket           SumOrderQuantity SumUnitPrice
--------------------------- ---------------- ---------------------
2011-01-03 00:00:00.0000000 21               65589.7546
2011-01-10 00:00:00.0000000 27               89938.5464
2011-01-17 00:00:00.0000000 31               104404.9064
2011-01-24 00:00:00.0000000 36               118525.6846
2011-01-31 00:00:00.0000000 39               123555.431
2011-02-07 00:00:00.0000000 35               109342.351
2011-02-14 00:00:00.0000000 32               107804.8964
2011-02-21 00:00:00.0000000 37               119456.3428
2011-02-28 00:00:00.0000000 9                28968.6982
```  

#### <a name="specifying-scalar-system-function-as-date"></a>Especificar la función de sistema escalar como date

En este ejemplo se especifica `SYSDATETIME` para *date*. El valor devuelto exacto depende del día y la hora de ejecución de la instrucción:
  
```sql
SELECT Date_Bucket(wk, 10, SYSDATETIME());  
```  

El conjunto de resultados es el siguiente:

```sql
---------------------------
2020-03-02 00:00:00.0000000

(1 row affected)
```  

#### <a name="specifying-scalar-subqueries-and-scalar-functions-as-number-and-date"></a>Especificar subconsultas y funciones escalares como number y date

En este ejemplo se usan subconsultas escalares, `MAX(OrderDate)`, como argumentos para *number* y *date*. `(SELECT top 1 CustomerKey FROM dbo.DimCustomer where GeographyKey > 100)` actúa como argumento artificial para el parámetro "number", para mostrar cómo se selecciona un argumento *number* de una lista de valores.
  
```sql
SELECT DATE_BUCKET(week,(SELECT top 1 CustomerKey FROM dbo.DimCustomer where GeographyKey > 100),  
    (SELECT MAX(OrderDate) FROM dbo.FactInternetSales));  
```  
  
#### <a name="specifying-numeric-expressions-and-scalar-system-functions-as-number-and-date"></a>Especificar expresiones numéricas y funciones del sistema escalares como number y date

En este ejemplo se usa una expresión numérica ((10/2)) y funciones de sistema escalares (SYSDATETIME) como argumentos para number y date.
  
```sql
SELECT Date_Bucket(week,(10/2), SYSDATETIME());
```

#### <a name="specifying-an-aggregate-window-function-as-number"></a>Especificar una función de ventana agregada como number

En este ejemplo se usa una función de ventana agregada como argumento para *number*.
  
```sql
Select 
    DISTINCT DATE_BUCKET(day, 30, Cast([shipdate] as datetime2)) as DateBucket,
    First_Value([SalesOrderNumber]) OVER (Order by DATE_BUCKET(day, 30, Cast([shipdate] as datetime2))) as First_Value_In_Bucket,
    Last_Value([SalesOrderNumber]) OVER (Order by DATE_BUCKET(day, 30, Cast([shipdate] as datetime2))) as Last_Value_In_Bucket
    from [dbo].[FactInternetSales]
Where ShipDate between '2011-01-03 00:00:00.000' and '2011-02-28 00:00:00.000'
order by DateBucket
GO  
``` 
### <a name="c-using-a-non-default-origin-value"></a>C. Uso de un valor de origen no predeterminado

En este ejemplo se usa un valor de origen no predeterminado para generar los cubos de fecha. 

```sql
declare @date datetime2 = '2020-06-15 21:22:11'
declare @origin datetime2 = '2019-01-01 00:00:00'
Select DATE_BUCKET(hh, 2, @date, @origin)
```

## <a name="see-also"></a>Consulte también

[CAST y CONVERT &#40;Transact-SQL&#41;](/sql/t-sql/functions/cast-and-convert-transact-sql/)
