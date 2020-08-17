---
title: Cláusula OFFSET LIMIT en Azure Cosmos DB
description: Obtenga información sobre cómo usar la cláusula OFFSET LIMIT para omitir y tomar algunos valores concretos al realizar consultas en Azure Cosmos DB
author: timsander1
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 07/29/2020
ms.author: tisande
ms.openlocfilehash: 9342c0c10ed4f7287716d21823fddfe992e0568f
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87432712"
---
# <a name="offset-limit-clause-in-azure-cosmos-db"></a>Cláusula OFFSET LIMIT en Azure Cosmos DB

La cláusula OFFSET LIMIT es una cláusula opcional para omitir y luego tomar cierto número de valores de la consulta. El recuento de OFFSET y el recuento de LIMIT son necesarios en la cláusula OFFSET LIMIT.

Cuando OFFSET LIMIT se usa junto con una cláusula ORDER BY, el conjunto de resultados se genera mediante una operación Skip y Take en los valores ordenados. Si no se usa ninguna cláusula ORDER BY, se producirá un error en un orden determinista de valores.

## <a name="syntax"></a>Sintaxis
  
```sql  
OFFSET <offset_amount> LIMIT <limit_amount>
```  
  
## <a name="arguments"></a>Argumentos

- `<offset_amount>`

   Especifica el número entero de elementos que los resultados de la consulta deben omitir.

- `<limit_amount>`
  
   Especifica el número entero de elementos que los resultados de la consulta deben incluir.

## <a name="remarks"></a>Observaciones
  
  Se necesitan los valores de `OFFSET` y `LIMIT` en la cláusula `OFFSET LIMIT`. Si se usa una cláusula `ORDER BY` opcional, se genera el conjunto de resultados mediante la omisión de los valores ordenados. En caso contrario, la consulta devolverá un orden fijo de valores.

  El cargo por RU de una consulta con `OFFSET LIMIT` aumentará a medida que aumente el número de términos que se van a desplazar. En el caso de las consultas que tienen [varias páginas de resultados](sql-query-pagination.md), normalmente se recomienda usar [tokens de continuación](sql-query-pagination.md#continuation-tokens). Los tokens de continuación son un "marcador" del lugar donde se puede reanudar la consulta más adelante. Si usa `OFFSET LIMIT`, no hay "marcador". Si desea devolver la página siguiente de la consulta, tendría que empezar desde el principio.
  
  Debe usar `OFFSET LIMIT` para los casos en los que quiera omitir elementos completamente y guardar los recursos del cliente. Por ejemplo, use `OFFSET LIMIT` si desea ir al resultado número 1000 de la consulta y no necesita ver los resultados 1 a 999. En el back-end, `OFFSET LIMIT` sigue cargando todos los elementos, incluidos los que se omiten. La ventaja en cuanto al rendimiento es un ahorro en los recursos de cliente porque se evita procesar elementos innecesarios.

## <a name="examples"></a>Ejemplos

Por ejemplo, he aquí una consulta que omite el primer valor y devuelve el segundo (en el orden del nombre de la ciudad de residencia):

```sql
    SELECT f.id, f.address.city
    FROM Families f
    ORDER BY f.address.city
    OFFSET 1 LIMIT 1
```

Los resultados son:

```json
    [
      {
        "id": "AndersenFamily",
        "city": "Seattle"
      }
    ]
```

Esta es una consulta que omite el primer valor y devuelve el segundo (sin ordenar):

```sql
    SELECT f.id, f.address.city
    FROM Families f
    OFFSET 1 LIMIT 1
```

Los resultados son:

```json
    [
      {
        "id": "WakefieldFamily",
        "city": "Seattle"
      }
    ]
```

## <a name="next-steps"></a>Pasos siguientes

- [Introducción](sql-query-getting-started.md)
- [Cláusula SELECT](sql-query-select.md)
- [Cláusula ORDER BY](sql-query-order-by.md)
