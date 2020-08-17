---
title: DateTimeDiff en el lenguaje de consulta de Azure Cosmos DB
description: Aprenda sobre la función del sistema DateTimeDiff en Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 07/09/2020
ms.author: tisande
ms.custom: query-reference
ms.openlocfilehash: b90b45072128252e8abc22d3422c84c813808119
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87446380"
---
# <a name="datetimediff-azure-cosmos-db"></a>DateTimeDiff (Azure Cosmos DB)

Devuelve el recuento (como valor entero con signo) de los límites de DateTimePart especificados que se han cruzado entre los valores *StartDate* y *EndDate* especificados.
  
## <a name="syntax"></a>Sintaxis
  
```sql
DateTimeDiff (<DateTimePart> , <StartDate> , <EndDate>)
```

## <a name="arguments"></a>Argumentos
  
*DateTimePart*  
   La parte de la fecha a la que DateTimeAdd agrega un número entero. En esta tabla se enumeran todos los argumentos válidos de DateTimePart.

| DateTimePart | abreviaturas        |
| ------------ | -------------------- |
| Year         | "year", "yyyy", "yy" |
| Month        | "month", "mm", "m"   |
| Día          | "day", "dd", "d"     |
| Hour         | "hour", "hh"         |
| Minute       | "minute", "mi", "n"  |
| Segundo       | "second", "ss", "s"  |
| Millisecond  | "millisecond", "ms"  |
| Microsecond  | "microsecond", "mcs" |
| Nanosecond   | "nanosecond", "ns"   |

*StartDate*  
    Valor de cadena de fecha y hora UTC ISO 8601 con el formato `YYYY-MM-DDThh:mm:ss.fffffffZ`, donde:
  
  |Formato|Descripción|
  |-|-|
  |AAAA|año con cuatro dígitos|
  |MM|mes con dos dígitos (01 = enero, etc.)|
  |DD|día con dos dígitos del mes (de 01 a 31)|
  |T|indica el comienzo de los elementos de hora|
  |hh|hora con dos dígitos (de 00 a 23)|
  |MM|minutos con dos dígitos (de 00 a 59)|
  |ss|segundos con dos dígitos (de 00 a 59)|
  |.fffffff|fracciones de segundos de siete dígitos|
  |Z|Indicador de UTC (hora universal coordinada)||
  
  Para más información sobre el formato ISO 8601, consulte [ISO_8601](https://en.wikipedia.org/wiki/ISO_8601)

*EndDate*  
   Valor de cadena de fecha y hora UTC ISO 8601 con el formato `YYYY-MM-DDThh:mm:ss.fffffffZ`

## <a name="return-types"></a>Tipos de valores devueltos

Devuelve un valor entero con signo.

## <a name="remarks"></a>Observaciones

DateTimeDiff devolverá `undefined` por los siguientes motivos:

- El valor DateTimePart especificado no es válido.
- StartDate o EndDate no es un valor DateTime ISO 8601 válido

DateTimeDiff siempre devolverá un valor entero con signo y es una medida del número de límites de DateTimePart que se han cruzado, no de la medición del intervalo de tiempo.

## <a name="examples"></a>Ejemplos
  
En el ejemplo siguiente se calcula el número de límites de día cruzados entre `2020-01-01T01:02:03.1234527Z` y `2020-01-03T01:02:03.1234567Z`.

```sql
SELECT DateTimeDiff("day", "2020-01-01T01:02:03.1234527Z", "2020-01-03T01:02:03.1234567Z") AS DifferenceInDays
```

```json
[
    {
        "DifferenceInDays": 2
    }
]
```  

En el ejemplo siguiente se calcula el número de límites de año cruzados entre `2028-01-01T01:02:03.1234527Z` y `2020-01-03T01:02:03.1234567Z`.

```sql
SELECT DateTimeDiff("yyyy", "2028-01-01T01:02:03.1234527Z", "2020-01-03T01:02:03.1234567Z") AS DifferenceInYears
```

```json
[
    {
        "DifferenceInYears": -8
    }
]
```

En el ejemplo siguiente se calcula el número de límites de hora cruzados entre `2020-01-01T01:00:00.1234527Z` y `2020-01-01T01:59:59.1234567Z`. Aunque estos valores de DateTime tienen una diferencia mayor a 0,99 horas, `DateTimeDiff` devuelve 0 porque no se cruzó ningún límite de hora.

```sql
SELECT DateTimeDiff("hh", "2020-01-01T01:00:00.1234527Z", "2020-01-01T01:59:59.1234567Z") AS DifferenceInHours
```

```json
[
    {
        "DifferenceInHours": 0
    }
]
```

## <a name="next-steps"></a>Pasos siguientes

- [Funciones de fecha y hora (Azure Cosmos DB)](sql-query-date-time-functions.md)
- [Funciones del sistema (Azure Cosmos DB)](sql-query-system-functions.md)
- [Introducción a Azure Cosmos DB](introduction.md)
