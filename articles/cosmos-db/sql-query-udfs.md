---
title: Funciones definidas por el usuario (UDF) en Azure Cosmos DB
description: Obtenga información sobre las funciones definidas por el usuario en Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 04/09/2020
ms.author: tisande
ms.custom: devx-track-javascript
ms.openlocfilehash: 9be2a6c4497806370c4c53cbd41d9f48779f8472
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87419757"
---
# <a name="user-defined-functions-udfs-in-azure-cosmos-db"></a>Funciones definidas por el usuario (UDF) en Azure Cosmos DB

La API de SQL proporciona compatibilidad con las funciones definidas por el usuario (UDF). Con las funciones definidas por el usuario escalares, puede pasar ningún argumento o muchos, y devolver un resultado con un único argumento. La API comprueba que cada argumento sea un valor JSON válido.  

## <a name="udf-use-cases"></a>Casos de uso de UDF

La API amplía la sintaxis de SQL para admitir una lógica de aplicación personalizada con funciones definidas por el usuario. Puede registrar las funciones definidas por el usuario con la API de SQL y hacer referencia a ellas en las consultas SQL. A diferencia de los procedimientos almacenados y los desencadenadores, las UDF son de solo lectura.

Con las UDF, puede extender el lenguaje de consulta de Azure Cosmos DB. Las UDF son una excelente manera de expresar una lógica de negocios compleja en la proyección de una consulta.

Sin embargo, se recomienda evitar las UDF cuando:

- Ya existe una [función del sistema](sql-query-system-functions.md) equivalente en Azure Cosmos DB. Las funciones del sistema siempre usarán menos RU que la UDF equivalente.
- La UDF es el único filtro de la cláusula `WHERE` de la consulta. Las UDF no usan el índice para que la evaluación de la UDF requiera la carga de documentos. La combinación de predicados de filtro adicionales que usan el índice, junto con una UDF, en la cláusula `WHERE` reducirá el número de documentos procesados por la UDF.

Si debe usar la misma UDF varias veces en una consulta, debe hacer referencia a la UDF en una [subconsulta](sql-query-subquery.md#evaluate-once-and-reference-many-times), lo que le permite usar una expresión JOIN para evaluar la UDF una vez, pero puede hacer referencia a ella muchas veces.

## <a name="examples"></a>Ejemplos

En el ejemplo siguiente se registra una función definida por el usuario en un contenedor de elementos en la base de datos de Cosmos. El ejemplo crea una función definida por el usuario cuyo nombre es `REGEX_MATCH`. Acepta dos valores de cadena JSON, `input` y `pattern`, y comprueba si el primero coincide con el patrón especificado en el segundo mediante la función `string.match()` de JavaScript.

```javascript
       UserDefinedFunction regexMatchUdf = new UserDefinedFunction
       {
           Id = "REGEX_MATCH",
           Body = @"function (input, pattern) {
                      return input.match(pattern) !== null;
                   };",
       };

       UserDefinedFunction createdUdf = client.CreateUserDefinedFunctionAsync(
           UriFactory.CreateDocumentCollectionUri("myDatabase", "families"),
           regexMatchUdf).Result;  
```

Ahora, use esta función definida por el usuario en la proyección de una consulta. Debe calificar las funciones definidas por el usuario con el prefijo que distingue mayúsculas de minúsculas `udf.` al llamarlas desde las consultas.

```sql
    SELECT udf.REGEX_MATCH(Families.address.city, ".*eattle")
    FROM Families
```

Los resultados son:

```json
    [
      {
        "$1": true
      },
      {
        "$1": false
      }
    ]
```

Puede usar la función definida por el usuario calificada con el prefijo `udf.` dentro de un filtro, como se muestra en el ejemplo siguiente:

```sql
    SELECT Families.id, Families.address.city
    FROM Families
    WHERE udf.REGEX_MATCH(Families.address.city, ".*eattle")
```

Los resultados son:

```json
    [{
        "id": "AndersenFamily",
        "city": "Seattle"
    }]
```

Básicamente, las funciones definidas por el usuario son expresiones escalares válidas que puede usar tanto en proyecciones como en filtros.

Para ampliar la capacidad de las funciones definidas por el usuario, echemos un vistazo a otro ejemplo con lógica condicional:

```javascript
       UserDefinedFunction seaLevelUdf = new UserDefinedFunction()
       {
           Id = "SEALEVEL",
           Body = @"function(city) {
                   switch (city) {
                       case 'Seattle':
                           return 520;
                       case 'NY':
                           return 410;
                       case 'Chicago':
                           return 673;
                       default:
                           return -1;
                    }"
            };

            UserDefinedFunction createdUdf = await client.CreateUserDefinedFunctionAsync(
                UriFactory.CreateDocumentCollectionUri("myDatabase", "families"),
                seaLevelUdf);
```

En el ejemplo siguiente se usa la función definida por el usuario:

```sql
    SELECT f.address.city, udf.SEALEVEL(f.address.city) AS seaLevel
    FROM Families f
```

Los resultados son:

```json
     [
      {
        "city": "Seattle",
        "seaLevel": 520
      },
      {
        "city": "NY",
        "seaLevel": 410
      }
    ]
```

Si las propiedades a las que hacen referencia los parámetros de la función no están disponibles en el valor JSON, el parámetro se considera indefinido y la invocación de la función se omite. De forma similar, si el resultado de la función definida por el usuario es indefinido, no se incluye en el resultado.

Como se muestra en los ejemplos anteriores, las funciones definidas por el usuario integran la eficacia del lenguaje JavaScript con la API de SQL. Las funciones definidas por el usuario proporcionan una interfaz programable enriquecida para lograr una lógica condicional de procedimientos compleja con la ayuda de funcionalidades de tiempo de ejecución de JavaScript integradas. La API de SQL proporciona a las funciones definidas por el usuario los argumentos para cada elemento de origen en la fase actual de procesamiento de las cláusulas WHERE o SELECT. El resultado se incorpora a la perfección en la canalización de la ejecución general. En resumen, las funciones definidas por el usuario son excelentes herramientas para lograr una lógica de negocios compleja como parte de las consultas.

## <a name="next-steps"></a>Pasos siguientes

- [Introducción a Azure Cosmos DB](introduction.md)
- [System functions](sql-query-system-functions.md) (Funciones del sistema)
- [Aggregates](sql-query-aggregates.md) (Agregados)