---
title: REPLICATE en el lenguaje de consulta de Azure Cosmos DB
description: Obtenga información acerca de la función del sistema de SQL REPLICATE en Azure Cosmos DB.
author: ginamr
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 03/03/2020
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: aea29cfff6b3827cfb9169722e48120e3a5a3709
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88794316"
---
# <a name="replicate-azure-cosmos-db"></a>REPLICATE (Azure Cosmos DB)
 Repite un valor de cadena un número especificado de veces.
  
## <a name="syntax"></a>Sintaxis
  
```sql
REPLICATE(<str_expr>, <num_expr>)
```  
  
## <a name="arguments"></a>Argumentos
  
*str_expr*  
   Es una expresión de cadena.
  
*num_expr*  
   Es una expresión numérica. Si el valor *num_expr* es negativo o no finito, el resultado es indefinido.
  
## <a name="return-types"></a>Tipos de valores devueltos
  
  Devuelve una expresión de cadena.
  
## <a name="remarks"></a>Observaciones

  La longitud máxima del resultado es de 10 000 caracteres, es decir, (length(*str_expr*)  *  *num_expr*) <= 10 000. Esta función del sistema no usará el índice.

## <a name="examples"></a>Ejemplos
  
  En el ejemplo siguiente se muestra cómo usar `REPLICATE` en una consulta.
  
```sql
SELECT REPLICATE("a", 3) AS replicate
```  
  
 El conjunto de resultados es el siguiente:
  
```json
[{"replicate": "aaa"}]
```  

## <a name="next-steps"></a>Pasos siguientes

- [Funciones de cadena Azure Cosmos DB](sql-query-string-functions.md)
- [Funciones del sistema (Azure Cosmos DB)](sql-query-system-functions.md)
- [Introducción a Azure Cosmos DB](introduction.md)
