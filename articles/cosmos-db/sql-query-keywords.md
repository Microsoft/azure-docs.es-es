---
title: Palabras clave de SQL para Azure Cosmos DB
description: Obtenga información sobre las palabras clave de SQL para Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 07/29/2020
ms.author: tisande
ms.openlocfilehash: f00e757f9b51da850c49924f6ae49bf00c9c53d1
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87496688"
---
# <a name="keywords-in-azure-cosmos-db"></a>Palabras clave en Azure Cosmos DB

En este artículo se detallan las palabras clave que se pueden usar en consultas SQL de Azure Cosmos DB.

## <a name="between"></a>BETWEEN

Puede usar la palabra clave `BETWEEN` para expresar consultas en intervalos de cadenas o valores numéricos. Por ejemplo, la consulta siguiente devuelve todos los elementos de la familia en los que el curso del primer hijo se encuentre entre 1 y 5, ambos incluidos.

```sql
    SELECT *
    FROM Families.children[0] c
    WHERE c.grade BETWEEN 1 AND 5
```

También puede usar la palabra clave `BETWEEN` en la cláusula `SELECT`, como en el ejemplo siguiente.

```sql
    SELECT (c.grade BETWEEN 0 AND 10)
    FROM Families.children[0] c
```

En la API de SQL, a diferencia de SQL ANSI, puede expresar consultas en intervalos en propiedades de tipos combinados. Por ejemplo, `grade` podría ser un número, como `5`, en algunos elementos y una cadena, como `grade4`, en otros. En estos casos, al igual que en JavaScript, una comparación entre los dos tipos distintos da como resultado `Undefined`, por lo que el elemento se omite.

## <a name="distinct"></a>DISTINCT

La palabra clave `DISTINCT` elimina los duplicados en la proyección de la consulta.

En este ejemplo, la consulta proyecta los valores para cada apellido:

```sql
SELECT DISTINCT VALUE f.lastName
FROM Families f
```

Los resultados son:

```json
[
    "Andersen"
]
```

También puede proyectar objetos únicos. En este caso, el campo lastName no existe en uno de los dos documentos, por lo que la consulta devuelve un objeto vacío.

```sql
SELECT DISTINCT f.lastName
FROM Families f
```

Los resultados son:

```json
[
    {
        "lastName": "Andersen"
    },
    {}
]
```

`DISTINCT` también se puede usar en la proyección dentro de una subconsulta:

```sql
SELECT f.id, ARRAY(SELECT DISTINCT VALUE c.givenName FROM c IN f.children) as ChildNames
FROM f
```

Esta consulta proyecta una matriz que contiene el valor de givenName de cada hijo con los duplicados quitados. Esta matriz tiene establecido un alias como ChildNames y se proyecta en la consulta externa.

Los resultados son:

```json
[
    {
        "id": "AndersenFamily",
        "ChildNames": []
    },
    {
        "id": "WakefieldFamily",
        "ChildNames": [
            "Jesse",
            "Lisa"
        ]
    }
]
```

No se admiten las consultas con una función del sistema agregada y una subconsulta con `DISTINCT`. Por ejemplo, la siguiente consulta no se admite:

```sql
SELECT COUNT(1) FROM (SELECT DISTINCT f.lastName FROM f)
```

## <a name="in"></a>IN

Use la palabra clave IN para comprobar si un valor especificado coincide con algún valor de una lista. Por ejemplo, la consulta siguiente devuelve todos los elementos de la familia donde `id` es `WakefieldFamily` o `AndersenFamily`.

```sql
    SELECT *
    FROM Families
    WHERE Families.id IN ('AndersenFamily', 'WakefieldFamily')
```

El ejemplo siguiente devuelve todos los elementos cuyo estado sea cualquiera de los valores especificados:

```sql
    SELECT *
    FROM Families
    WHERE Families.address.state IN ("NY", "WA", "CA", "PA", "OH", "OR", "MI", "WI", "MN", "FL")
```

La API de SQL proporciona compatibilidad con la [iteración en las matrices JSON](sql-query-object-array.md#Iteration), donde se ha agregado una nueva construcción mediante la palabra clave IN en el origen FROM.

Si incluye la clave de partición en el filtro `IN`, la consulta se filtrará automáticamente solo por las particiones pertinentes.

## <a name="top"></a>TOP

La palabra clave TOP devuelve los primeros `N` resultados de la consulta en un orden indefinido. Como procedimiento recomendado, use TOP con la cláusula `ORDER BY` para limitar los resultados a los primeros `N` valores ordenados. La combinación de estas dos cláusulas es la única manera de indicar de forma previsible las filas a las que TOP afecta.

Puede usar TOP con un valor constante, como se muestra en el ejemplo anterior, o con un valor variable usando consultas parametrizadas.

```sql
    SELECT TOP 1 *
    FROM Families f
```

Los resultados son:

```json
    [{
        "id": "AndersenFamily",
        "lastName": "Andersen",
        "parents": [
           { "firstName": "Thomas" },
           { "firstName": "Mary Kay"}
        ],
        "children": [
           {
               "firstName": "Henriette Thaulow", "gender": "female", "grade": 5,
               "pets": [{ "givenName": "Fluffy" }]
           }
        ],
        "address": { "state": "WA", "county": "King", "city": "Seattle" },
        "creationDate": 1431620472,
        "isRegistered": true
    }]
```

## <a name="next-steps"></a>Pasos siguientes

- [Introducción](sql-query-getting-started.md)
- [Combinaciones](sql-query-join.md)
- [Subconsultas](sql-query-subquery.md)