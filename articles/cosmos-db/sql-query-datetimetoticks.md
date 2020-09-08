---
title: DateTimeToTicks en el lenguaje de consulta de Azure Cosmos DB
description: Obtenga información acerca de la función del sistema de SQL DateTimeToTicks en Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 08/18/2020
ms.author: tisande
ms.custom: query-reference
ms.openlocfilehash: 2e2c9e8f2bf0d4760bf030fb19a90737cdb54525
ms.sourcegitcommit: d661149f8db075800242bef070ea30f82448981e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88605146"
---
# <a name="datetimetoticks-azure-cosmos-db"></a>DateTimeToTicks (Azure Cosmos DB)

Convierte el valor de DateTime especificado en tics. Un solo tic representa cien nanosegundos o una diezmillonésima parte de un segundo. 

## <a name="syntax"></a>Sintaxis
  
```sql
DateTimeToTicks (<DateTime>)
```

## <a name="arguments"></a>Argumentos
  
*DateTime*  
   Valor de cadena de fecha y hora UTC ISO 8601 con el formato `YYYY-MM-DDThh:mm:ss.fffffffZ`

## <a name="return-types"></a>Tipos de valores devueltos

Devuelve un valor numérico con signo, el número actual de tics de 100 nanosegundos que han transcurrido desde la época de Unix. En otras palabras, DateTimeToTicks devuelve el número de tics de 100 nanosegundos que han transcurrido desde las 00:00:00 del jueves, 1 de enero de 1970.

## <a name="remarks"></a>Observaciones

DateTimeDateTimeToTicks devolverá `undefined` si DateTime no es una DateTime ISO 8601 válida

Esta función del sistema no usará el índice.

## <a name="examples"></a>Ejemplos

Este es un ejemplo que devuelve el número de tics:

```sql
SELECT DateTimeToTicks("2020-01-02T03:04:05.6789123Z") AS Ticks
```

```json
[
    {
        "Ticks": 15779342456789124
    }
]
```

Este es un ejemplo que devuelve el número de tics sin especificar el número de fracciones de segundo:

```sql
SELECT DateTimeToTicks("2020-01-02T03:04:05Z") AS Ticks
```

```json
[
    {
        "Ticks": 15779342450000000
    }
]
```

## <a name="next-steps"></a>Pasos siguientes

- [Funciones de fecha y hora (Azure Cosmos DB)](sql-query-date-time-functions.md)
- [Funciones del sistema (Azure Cosmos DB)](sql-query-system-functions.md)
- [Introducción a Azure Cosmos DB](introduction.md)
