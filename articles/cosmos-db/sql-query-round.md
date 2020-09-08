---
title: ROUND en el lenguaje de consulta de Azure Cosmos DB
description: Obtenga información acerca de la función del sistema de SQL ROUND en Azure Cosmos DB.
author: ginamr
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 09/13/2019
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: 9712aedd3d6748f3bceea67a3270b6c080cc16f2
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88794278"
---
# <a name="round-azure-cosmos-db"></a>ROUND (Azure Cosmos DB)
 Devuelve un valor numérico, redondeado al valor entero más cercano.  
  
## <a name="syntax"></a>Sintaxis
  
```sql
ROUND(<numeric_expr>)  
```  
  
## <a name="arguments"></a>Argumentos
  
*numeric_expr*  
   Es una expresión numérica.  
  
## <a name="return-types"></a>Tipos de valores devueltos
  
  Devuelve una expresión numérica.  
  
## <a name="remarks"></a>Observaciones
  
  La operación de redondeo realizada sigue el redondeo de punto medio alejado del cero. Si la entrada es una expresión numérica que se encuentra exactamente entre dos enteros, el resultado será el valor entero más cercano alejado de cero. Esta función del sistema se beneficiará de un [índice de intervalo](index-policy.md#includeexclude-strategy).
  
  |<numeric_expr>|Redondeo|
  |-|-|
  |-6,5000|-7|
  |-0,5|-1|
  |0.5|1|
  |6,5000|7||
  
## <a name="examples"></a>Ejemplos
  
  En el ejemplo siguiente se redondean los siguientes números positivos y negativos al entero más próximo.  
  
```sql
SELECT ROUND(2.4) AS r1, ROUND(2.6) AS r2, ROUND(2.5) AS r3, ROUND(-2.4) AS r4, ROUND(-2.6) AS r5  
```  
  
  El conjunto de resultados es el siguiente:  
  
```json
[{r1: 2, r2: 3, r3: 3, r4: -2, r5: -3}]  
```  

## <a name="next-steps"></a>Pasos siguientes

- [Funciones matemáticas (Azure Cosmos DB)](sql-query-mathematical-functions.md)
- [Funciones del sistema (Azure Cosmos DB)](sql-query-system-functions.md)
- [Introducción a Azure Cosmos DB](introduction.md)
