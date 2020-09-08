---
title: LOG10 en el lenguaje de consulta de Azure Cosmos DB
description: Obtenga información sobre la función del sistema SQL LOG10 en Azure Cosmos DB para devolver el logaritmo en base 10 de la expresión numérica especificada.
author: ginamr
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 09/13/2019
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: 6869abb3040feb6431d60799536c9986c6ccb954
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88798251"
---
# <a name="log10-azure-cosmos-db"></a>LOG10 (Azure Cosmos DB)
 Devuelve el logaritmo de base 10 de la expresión numérica especificada.  
  
## <a name="syntax"></a>Sintaxis
  
```sql
LOG10 (<numeric_expr>)  
```  
  
## <a name="arguments"></a>Argumentos
  
*numeric_expression*  
   Es una expresión numérica.  
  
## <a name="return-types"></a>Tipos de valores devueltos
  
  Devuelve una expresión numérica.  
  
## <a name="remarks"></a>Observaciones
  
  Las funciones LOG10 y POWER están relacionadas inversamente entre sí. Por ejemplo, 10 ^ LOG10 (n) = n. Esta función del sistema no usará el índice.
  
## <a name="examples"></a>Ejemplos
  
  En el ejemplo siguiente se declara una variable y se devuelve el valor de LOG10 de la variable especificada (100).  
  
```sql
SELECT LOG10(100) AS log10 
```  
  
 El conjunto de resultados es el siguiente:  
  
```json
[{log10: 2}]  
```  

## <a name="next-steps"></a>Pasos siguientes

- [Funciones matemáticas (Azure Cosmos DB)](sql-query-mathematical-functions.md)
- [Funciones del sistema (Azure Cosmos DB)](sql-query-system-functions.md)
- [Introducción a Azure Cosmos DB](introduction.md)
