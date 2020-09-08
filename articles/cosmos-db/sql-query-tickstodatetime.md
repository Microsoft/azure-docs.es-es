---
title: TicksToDateTime en el lenguaje de consulta de Azure Cosmos DB
description: Aprenda sobre la función TicksToDateTime en Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 08/18/2020
ms.author: tisande
ms.custom: query-reference
ms.openlocfilehash: 89a8dba97725049b86fc6b38c09e0dd125bb48d1
ms.sourcegitcommit: d661149f8db075800242bef070ea30f82448981e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88608663"
---
# <a name="tickstodatetime-azure-cosmos-db"></a>TicksToDateTime (Azure Cosmos DB)

Convierte el valor de tics especificado en un valor de fecha y hora.
  
## <a name="syntax"></a>Sintaxis
  
```sql
TicksToDateTime (<Ticks>)
```

## <a name="arguments"></a>Argumentos

*Tics*  

Un valor numérico con signo, el número actual de tics de 100 nanosegundos que han transcurrido desde la época de Unix. Es decir, el número de tics de 100 nanosegundos que han transcurrido desde las 00:00:00 del jueves, 1 de enero de 1970.

## <a name="return-types"></a>Tipos de valores devueltos

Devuelve el valor de cadena de fecha y hora UTC ISO 8601 con el formato `YYYY-MM-DDThh:mm:ss.fffffffZ`, donde:
  
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

## <a name="remarks"></a>Observaciones

TicksToDateTime devolverá `undefined` si el valor de tics especificado no es válido.

## <a name="examples"></a>Ejemplos
  
En el ejemplo siguiente se convierten los tics en un valor fecha y hora:

```sql
SELECT TicksToDateTime(15943368134575530) AS DateTime
```

```json
[
    {
        "DateTime": "2020-07-09T23:20:13.4575530Z"
    }
]
```  

## <a name="next-steps"></a>Pasos siguientes

- [Funciones de fecha y hora (Azure Cosmos DB)](sql-query-date-time-functions.md)
- [Funciones del sistema (Azure Cosmos DB)](sql-query-system-functions.md)
- [Introducción a Azure Cosmos DB](introduction.md)
