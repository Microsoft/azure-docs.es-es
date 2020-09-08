---
title: DateTimePart en el lenguaje de consulta de Azure Cosmos DB
description: Obtenga información sobre la función del sistema de SQL DateTimePart en Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 08/14/2020
ms.author: tisande
ms.custom: query-reference
ms.openlocfilehash: 87663c18ddaa5da6740a0f54aa5f2812cbb06af8
ms.sourcegitcommit: 152c522bb5ad64e5c020b466b239cdac040b9377
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88226965"
---
# <a name="datetimepart-azure-cosmos-db"></a>DateTimePart (Azure Cosmos DB)

Devuelve el valor del elemento DateTimePart especificado entre la fecha y hora especificada.
  
## <a name="syntax"></a>Sintaxis
  
```sql
DateTimePart (<DateTimePart> , <DateTime>)
```

## <a name="arguments"></a>Argumentos
  
*DateTimePart*  
   Parte de la fecha para la que DateTimePart devolverá el valor. En esta tabla se enumeran todos los argumentos válidos de DateTimePart.

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

*DateTime*  
   Valor de cadena de fecha y hora UTC ISO 8601 con el formato `YYYY-MM-DDThh:mm:ss.fffffffZ`

## <a name="return-types"></a>Tipos de valores devueltos

Devuelve un valor entero positivo.

## <a name="remarks"></a>Observaciones

DateTimePart devolverá `undefined` por los siguientes motivos:

- El valor DateTimePart especificado no es válido.
- La fecha y hora no son válidas de acuerdo con ISO 8601.

Esta función del sistema no usará el índice.

## <a name="examples"></a>Ejemplos

Este es un ejemplo que devuelve el valor entero del mes:

```sql
SELECT DateTimePart("m", "2020-01-02T03:04:05.6789123Z") AS MonthValue
```

```json
[
    {
        "MonthValue": 1
    }
]
```

Este es un ejemplo que devuelve el número de microsegundos:

```sql
SELECT DateTimePart("mcs", "2020-01-02T03:04:05.6789123Z") AS MicrosecondsValue
```

```json
[
    {
        "MicrosecondsValue": 678912
    }
]
```

## <a name="next-steps"></a>Pasos siguientes

- [Funciones de fecha y hora (Azure Cosmos DB)](sql-query-date-time-functions.md)
- [Funciones del sistema (Azure Cosmos DB)](sql-query-system-functions.md)
- [Introducción a Azure Cosmos DB](introduction.md)
