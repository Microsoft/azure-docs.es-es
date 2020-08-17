---
title: Operadores de consultas SQL para Azure Cosmos DB
description: Obtenga información sobre los operadores de SQL, por ejemplo, los de igualdad, comparación y los operadores lógicos que admite Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 07/29/2020
ms.author: tisande
ms.openlocfilehash: dd1652781d7eae8beb400c52137a8f16891e2b2a
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87498844"
---
# <a name="operators-in-azure-cosmos-db"></a>Operadores en Azure Cosmos DB

En este artículo se detallan los diversos operadores admitidos por Azure Cosmos DB.

## <a name="equality-and-comparison-operators"></a>Operadores de igualdad y de comparación

En la siguiente tabla se muestra el resultado de las comparaciones de igualdad en la API de SQL entre dos tipos JSON cualesquiera.

| **Op** | **Undefined** | **Null** | **Boolean** | **Number** | **String** | **Object** | **Array** |
|---|---|---|---|---|---|---|---|
| **Undefined** | No definido | No definido | No definido | No definido | No definido | No definido | No definido |
| **Null** | No definido | **OK (CORRECTO)** | No definido | No definido | No definido | No definido | No definido |
| **Boolean** | No definido | No definido | **OK (CORRECTO)** | No definido | No definido | No definido | No definido |
| **Number** | No definido | No definido | No definido | **OK (CORRECTO)** | No definido | No definido | No definido |
| **String** | No definido | No definido | No definido | No definido | **OK (CORRECTO)** | No definido | No definido |
| **Object** | No definido | No definido | No definido | No definido | No definido | **OK (CORRECTO)** | No definido |
| **Array** | No definido | No definido | No definido | No definido | No definido | No definido | **OK (CORRECTO)** |

Para los operadores de comparación como `>`, `>=`, `!=`, `<` y `<=`, la comparación entre tipos o entre dos objetos o matrices genera `Undefined`.  

Si el resultado de la expresión escalar es `Undefined`, el elemento no se incluye en el resultado, porque `Undefined` no es igual a `true`.

Por ejemplo, la siguiente comparación de la consulta entre un número y un valor de cadena produce `Undefined`. Por lo tanto, el filtro no incluye ningún resultado.

```sql
SELECT *
FROM c
WHERE 7 = 'a'
```

## <a name="logical-and-or-and-not-operators"></a>Operadores lógicos (Y, O y NO)

Los operadores lógicos operan en valores booleanos. En las tablas siguientes se muestran las tablas de verdad lógica para estos operadores:

**operator OR**

Devuelve `true` cuando alguna de las condiciones es `true`.

|  | **True** | **False** | **Undefined** |
| --- | --- | --- | --- |
| **True** |True |True |True |
| **False** |True |False |No definido |
| **Undefined** |True |Sin definir |Sin definir |

**operator AND**

Devuelve `true` cuando ambas expresiones son `true`.

|  | **True** | **False** | **Undefined** |
| --- | --- | --- | --- |
| **True** |True |False |No definido |
| **False** |False |False |False |
| **Undefined** |No definido |False |No definido |

**operator NOT**

Invierte el valor de cualquier expresión booleana.

|  | **NOT** |
| --- | --- |
| **True** |False |
| **False** |True |
| **Undefined** |No definido |

**Prioridad de los operadores**

Los operadores lógicos `OR`, `AND` y `NOT` tienen el nivel de prioridad que se muestra a continuación:

| **Operador** | **Prioridad** |
| --- | --- |
| **NOT** |1 |
| **AND** |2 |
| **OR** |3 |

## <a name="-operator"></a>Operador *

El operador especial * proyecta el elemento completo tal cual. Al usarse, debe ser el único campo proyectado. Una consulta como `SELECT * FROM Families f` es válida; `SELECT VALUE * FROM Families f` y `SELECT *, f.id FROM Families f` no lo son.

## <a name="-and--operators"></a>? y ?? (operadores)

Puede usar el operador ternario (?) y el operador de combinación (??) para crear expresiones condicionales, al igual que en lenguajes de programación populares como C# y JavaScript.

Puede usar el operador ? para construir nuevas propiedades JSON sobre la marcha. Por ejemplo, la siguiente consulta clasifica los niveles de curso en `elementary` u `other`:

```sql
     SELECT (c.grade < 5)? "elementary": "other" AS gradeLevel
     FROM Families.children[0] c
```

Además, también puede anidar llamadas al operador ?, como en la consulta siguiente: 

```sql
    SELECT (c.grade < 5)? "elementary": ((c.grade < 9)? "junior": "high") AS gradeLevel
    FROM Families.children[0] c
```

Al igual que con otros operadores de consulta, el operador ? excluye elementos si faltan las propiedades a las que se hace referencia o si los tipos que se comparan son diferentes.

Use el operador ?? para comprobar de manera eficaz una propiedad en un elemento al consultar datos semiestructurados o de tipos mixtos. Por ejemplo, la consulta siguiente devuelve `lastName` si está presente, o `surname` si `lastName` no lo está.

```sql
    SELECT f.lastName ?? f.surname AS familyName
    FROM Families f
```

## <a name="next-steps"></a>Pasos siguientes

- [Ejemplos de .NET de Azure Cosmos DB](https://github.com/Azure/azure-cosmos-dotnet-v3)
- [Palabras clave](sql-query-keywords.md)
- [Cláusula SELECT](sql-query-select.md)
