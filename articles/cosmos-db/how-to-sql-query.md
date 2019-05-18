---
title: Consultas SQL para Azure Cosmos DB
description: Más información sobre la sintaxis SQL, los conceptos de base de datos y las consultas SQL para Azure Cosmos DB. Utiliza SQL como un lenguaje de consulta de JSON de Azure Cosmos DB.
author: markjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 05/06/2019
ms.author: mjbrown
ms.openlocfilehash: 4d1ef650a3f12d8b97cbad3e9aecf31c8b81a038
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65796159"
---
# <a name="sql-query-examples-for-azure-cosmos-db"></a>Ejemplos de consultas SQL para Azure Cosmos DB

Las cuentas de Azure Cosmos DB SQL API admiten al consultar elementos mediante el lenguaje de consulta estructurado (SQL) como lenguaje de consulta JSON. Los objetivos de diseño del lenguaje de consulta de Azure Cosmos DB son:

* Admite SQL, uno de los lenguajes de consulta más familiares y populares, en lugar de tener que inventar un nuevo lenguaje de consulta. SQL proporciona un modelo de programación formal para consultas enriquecidas a través de los elementos JSON.  

* Utilice el modelo de programación de JavaScript como base para el lenguaje de consulta. Sistema de tipos de JavaScript, la evaluación de expresiones y la invocación de función son las raíces de la API de SQL. Estas raíces proporcionan un modelo de programación natural para características como proyecciones relacionales, navegación jerárquica entre elementos JSON, autocombinaciones, consultas espaciales e invocación de funciones definidas por el usuario (UDF) escritas íntegramente en JavaScript.

Este artículo le guiará a través de algunos ejemplos de consultas SQL en elementos sencillos de JSON. Para obtener más información sobre la sintaxis del lenguaje SQL de Azure Cosmos DB, vea [referencia de sintaxis SQL](sql-api-query-reference.md).

## <a id="GettingStarted"></a>Empezar a trabajar con consultas SQL

En la cuenta de Cosmos DB de SQL API, cree un contenedor denominado `Families`. Cree dos elementos sencillos de JSON en el contenedor y ejecutar algunas consultas sencillas.

### <a name="create-json-items"></a>Crear elementos de JSON

El siguiente código crea dos elementos JSON simple sobre familias. Los elementos JSON simple para la familia Andersen e Wakefield incluyen elementos primarios, secundarios y sus mascotas, dirección e información de registro. El primer elemento tiene cadenas, números, valores booleanos, matrices y propiedades anidadas.


```json
{
  "id": "AndersenFamily",
  "lastName": "Andersen",
  "parents": [
     { "firstName": "Thomas" },
     { "firstName": "Mary Kay"}
  ],
  "children": [
     {
         "firstName": "Henriette Thaulow",
         "gender": "female",
         "grade": 5,
         "pets": [{ "givenName": "Fluffy" }]
     }
  ],
  "address": { "state": "WA", "county": "King", "city": "Seattle" },
  "creationDate": 1431620472,
  "isRegistered": true
}
```

El segundo elemento utiliza `givenName` y `familyName` en lugar de `firstName` y `lastName`.

```json
{
  "id": "WakefieldFamily",
  "parents": [
      { "familyName": "Wakefield", "givenName": "Robin" },
      { "familyName": "Miller", "givenName": "Ben" }
  ],
  "children": [
      {
        "familyName": "Merriam",
        "givenName": "Jesse",
        "gender": "female", 
        "grade": 1,
        "pets": [
            { "givenName": "Goofy" },
            { "givenName": "Shadow" }
        ]
      },
      { 
        "familyName": "Miller",
         "givenName": "Lisa",
         "gender": "female",
         "grade": 8 }
  ],
  "address": { "state": "NY", "county": "Manhattan", "city": "NY" },
  "creationDate": 1431620462,
  "isRegistered": false
}
```

### <a name="query-the-json-items"></a>Consultar los elementos JSON

Pruebe algunas consultas con los datos JSON para entender algunos aspectos clave del lenguaje de consulta SQL de Azure Cosmos DB.

La siguiente consulta devuelve los elementos donde la `id` campo coincidencias `AndersenFamily`. Puesto que es un `SELECT *` consulta, el resultado de la consulta es el elemento JSON completo. Para obtener más información sobre la sintaxis SELECT, vea [instrucción SELECT](sql-api-query-reference.md#select-query). 

```sql
    SELECT *
    FROM Families f
    WHERE f.id = "AndersenFamily"
```

Los resultados de consulta son: 

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

La siguiente consulta se vuelve a formatear la salida JSON en una forma diferente. La consulta proyecta un nuevo archivo JSON de `Family` objeto con dos campos seleccionados, `Name` y `City`, cuando la ciudad es el mismo que el estado. "Nueva York, NY" coincide con este caso.

```sql
    SELECT {"Name":f.id, "City":f.address.city} AS Family
    FROM Families f
    WHERE f.address.city = f.address.state
```

Los resultados de consulta son:

```json
    [{
        "Family": {
            "Name": "WakefieldFamily",
            "City": "NY"
        }
    }]
```

La siguiente consulta devuelve todos los nombres proporcionados de los elementos secundarios de la familia cuyo `id` coincide con `WakefieldFamily`, ordenada por ciudad.

```sql
    SELECT c.givenName
    FROM Families f
    JOIN c IN f.children
    WHERE f.id = 'WakefieldFamily'
    ORDER BY f.address.city ASC
```

Los resultados son:

```json
    [
      { "givenName": "Jesse" },
      { "givenName": "Lisa"}
    ]
```

Los ejemplos anteriores muestran varios aspectos del lenguaje de consulta de Cosmos DB:  

* Dado que las API de SQL funciona en valores JSON, trata de las entidades en forma de árbol en lugar de filas y columnas. Puede hacer referencia a los nodos de árbol a cualquier profundidad arbitraria, como `Node1.Node2.Node3…..Nodem`, de forma similar a la referencia de dos partes de `<table>.<column>` en ANSI SQL.

* Dado que el lenguaje de consulta funciona con datos sin esquema, el sistema de tipos debe enlazarse dinámicamente. La misma expresión podría generar diversos tipos en distintos elementos. El resultado de una consulta es un valor JSON válido, pero no está garantizado que sea de un esquema fijo.  

* Azure Cosmos DB solo admite elementos JSON estrictos. El sistema de tipos y las expresiones se restringen para tratar únicamente tipos JSON. Para obtener más información, consulte el [especificación de JSON](https://www.json.org/).  

* Un contenedor de Cosmos DB es una colección sin esquemas de elementos JSON. Las relaciones dentro y entre los elementos de contenedor se capturan implícitamente por contención, no por la clave principal y las relaciones de clave externas. Esta característica es importante para las combinaciones de elemento de intra descritas más adelante en este artículo.

## <a id="SelectClause"></a>Cláusula SELECT

Cada consulta consta de una cláusula SELECT y FROM opcional y las cláusulas WHERE, según los estándares ANSI SQL. Normalmente, se enumera el origen en la cláusula FROM y la cláusula WHERE aplica un filtro en el origen para recuperar un subconjunto de elementos de JSON. La cláusula SELECT, a continuación, proyecta los valores JSON solicitados en la lista de selección. Para obtener más información sobre la sintaxis, vea [instrucción SELECT](sql-api-query-reference.md#select-query).

Consulta de la siguiente instrucción SELECT devuelve de ejemplo `address` desde `Families` cuyo `id` coincide con `AndersenFamily`:

```sql
    SELECT f.address
    FROM Families f
    WHERE f.id = "AndersenFamily"
```

Los resultados son:

```json
    [{
      "address": {
        "state": "WA",
        "county": "King",
        "city": "Seattle"
      }
    }]
```

## <a id="EscapingReservedKeywords"></a>Descriptor de acceso de propiedad entre comillas
Puede tener acceso a las propiedades mediante el operador de propiedad entre comillas []. Por ejemplo, `SELECT c.grade` and `SELECT c["grade"]` son equivalentes. Esta sintaxis es útil para una propiedad que contiene espacios, caracteres especiales, o que tenga el mismo nombre que una palabra clave SQL o una palabra reservada de escape.

```sql
    SELECT f["lastName"]
    FROM Families f
    WHERE f["id"] = "AndersenFamily"
```

## <a name="nested-properties"></a>Propiedades anidadas

En el ejemplo siguiente se proyectan dos propiedades anidadas, `f.address.state` y `f.address.city`.

```sql
    SELECT f.address.state, f.address.city
    FROM Families f
    WHERE f.id = "AndersenFamily"
```

Los resultados son:

```json
    [{
      "state": "WA",
      "city": "Seattle"
    }]
```

## <a name="json-expressions"></a>Expresiones de JSON

La proyección también admite expresiones de JSON, tal como se muestra en el ejemplo siguiente:

```sql
    SELECT { "state": f.address.state, "city": f.address.city, "name": f.id }
    FROM Families f
    WHERE f.id = "AndersenFamily"
```

Los resultados son:

```json
    [{
      "$1": {
        "state": "WA",
        "city": "Seattle",
        "name": "AndersenFamily"
      }
    }]
```

En el ejemplo anterior, la cláusula SELECT se necesita para crear un objeto JSON y, puesto que el ejemplo proporciona ninguna clave, la cláusula utiliza el nombre de variable de argumentos implícitos `$1`. La siguiente consulta devuelve dos variables de argumentos implícitos: `$1` y `$2`.

```sql
    SELECT { "state": f.address.state, "city": f.address.city },
           { "name": f.id }
    FROM Families f
    WHERE f.id = "AndersenFamily"
```

Los resultados son:

```json
    [{
      "$1": {
        "state": "WA",
        "city": "Seattle"
      }, 
      "$2": {
        "name": "AndersenFamily"
      }
    }]
```

## <a id="ValueKeyword"></a>Palabra clave VALUE

La palabra clave VALUE proporciona una manera para devolver solo el valor JSON. Por ejemplo, la consulta se muestra a continuación devuelve la expresión escalar `"Hello World"` en lugar de `{$1: "Hello World"}`:

```sql
    SELECT VALUE "Hello World"
```

La consulta siguiente devuelve los valores JSON sin la `address` etiqueta:

```sql
    SELECT VALUE f.address
    FROM Families f
```

Los resultados son:

```json
    [
      {
        "state": "WA",
        "county": "King",
        "city": "Seattle"
      }, 
      {
        "state": "NY", 
        "county": "Manhattan",
        "city": "NY"
      }
    ]
```

El ejemplo siguiente muestra cómo devolver valores primitivos JSON (el nivel hoja del árbol JSON):


```sql
    SELECT VALUE f.address.state
    FROM Families f
```

Los resultados son:

```json
    [
      "WA",
      "NY"
    ]
```

## <a id="DistinctKeyword"></a>Palabra clave DISTINCT

La palabra clave DISTINCT elimina los duplicados en la proyección de la consulta.

```sql
SELECT DISTINCT VALUE f.lastName
FROM Families f
```

En este ejemplo, la consulta proyecta valores para cada apellido.

Los resultados son:

```json
[
    "Andersen"
]
```

También puede proyectar objetos únicos. En este caso, el campo Apellidos no existe en uno de los dos documentos, por lo que la consulta devuelve un objeto vacío.

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

También se puede usar DISTINCT en la proyección dentro de una subconsulta:

```sql
SELECT f.id, ARRAY(SELECT DISTINCT VALUE c.givenName FROM c IN f.children) as ChildNames
FROM f
```

Esta consulta proyecta una matriz que contiene givenName de cada elemento secundario con los duplicados quitados. Esta matriz tiene un alias como ChildNames y proyecta en la consulta externa.

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

## <a name="aliasing"></a>Establecimiento de alias

Se pueden convertir explícitamente alias valores en las consultas. Si una consulta tiene dos propiedades con el mismo nombre, utilice alias para cambiar el nombre de una o ambas de las propiedades por lo que elimina la ambigüedad en el resultado proyectado.

El AS de palabra clave utilizada para el alias es opcional, tal como se muestra en el ejemplo siguiente al proyectarse el segundo valor como `NameInfo`:

```sql
    SELECT 
           { "state": f.address.state, "city": f.address.city } AS AddressInfo,
           { "name": f.id } NameInfo
    FROM Families f
    WHERE f.id = "AndersenFamily"
```

Los resultados son:

```json
    [{
      "AddressInfo": {
        "state": "WA",
        "city": "Seattle"
      },
      "NameInfo": {
        "name": "AndersenFamily"
      }
    }]
```

## <a id="FromClause"></a>Cláusula FROM

FROM (`FROM <from_specification>`) cláusula es opcional, a menos que el origen se filtre o se proyecte posteriormente en la consulta. Para obtener más información sobre la sintaxis, vea [de sintaxis](sql-api-query-reference.md#bk_from_clause). Una consulta como `SELECT * FROM Families` enumera todo el `Families` contenedor. También puede usar la raíz de identificador especial para el contenedor en lugar de usar el nombre del contenedor.

La cláusula FROM exige las reglas siguientes por consulta:

* Se puede establecer un alias para el contenedor, como `SELECT f.id FROM Families AS f` o simplemente `SELECT f.id FROM Families f`. Aquí `f` es el alias para `Families`. COMO es el identificador de una palabra clave opcional al alias.  

* Una vez establecido un alias, no se puede enlazar el nombre de origen original. Por ejemplo, `SELECT Families.id FROM Families f` no es válido sintácticamente porque el identificador `Families` ha sido un alias y no puede resolverse.  

* Todas las propiedades que se hace referencia deben ser completos evitar cualquier enlace ambiguo en ausencia de adherencia estricta del esquema. Por ejemplo, `SELECT id FROM Families f` no es válido sintácticamente porque la propiedad `id` no está enlazado.

### <a name="get-subitems-by-using-the-from-clause"></a>Obtener subelementos mediante la cláusula FROM

La cláusula FROM puede reducir el origen a un subconjunto más pequeño. Para enumerar únicamente un subárbol en cada elemento, la subraíz puede convertirse en el origen, tal como se muestra en el ejemplo siguiente:

```sql
    SELECT *
    FROM Families.children
```

Los resultados son:

```json
    [
      [
        {
            "firstName": "Henriette Thaulow",
            "gender": "female",
            "grade": 5,
            "pets": [
              {
                  "givenName": "Fluffy"
              }
            ]
        }
      ],
      [
       {
            "familyName": "Merriam",
            "givenName": "Jesse",
            "gender": "female",
            "grade": 1
        },
        {
            "familyName": "Miller",
            "givenName": "Lisa",
            "gender": "female",
            "grade": 8
        }
      ]
    ]
```

La consulta anterior utiliza una matriz como origen, pero también puede usar un objeto como origen. La consulta se considera que cualquier valor JSON válido, que se define en el origen para su inclusión en el resultado. En el ejemplo siguiente se excluirían `Families` que no tienen un `address.state` valor.

```sql
    SELECT *
    FROM Families.address.state
```

Los resultados son:

```json
    [
      "WA",
      "NY"
    ]
```

## <a id="WhereClause"></a>Cláusula WHERE

La cláusula WHERE opcional (`WHERE <filter_condition>`) especifica las condiciones que deben cumplir los elementos de JSON de origen para que la consulta se incluirán en los resultados. Un elemento JSON debe evaluar las condiciones especificadas para `true` a tener en cuenta para el resultado. La capa de índice usa la cláusula WHERE para determinar el subconjunto más pequeño de elementos de origen que pueden formar parte del resultado. Para obtener más información sobre la sintaxis, vea [sintaxis WHERE](sql-api-query-reference.md#bk_where_clause).

La siguiente consulta de elementos de las solicitudes que contienen un `id` propiedad cuyo valor es `AndersenFamily`. Excluye cualquier elemento que no tiene un `id` propiedad o cuyo valor no coincide con `AndersenFamily`.

```sql
    SELECT f.address
    FROM Families f
    WHERE f.id = "AndersenFamily"
```

Los resultados son:

```json
    [{
      "address": {
        "state": "WA",
        "county": "King",
        "city": "Seattle"
      }
    }]
```

### <a name="scalar-expressions-in-the-where-clause"></a>Expresiones escalares en la cláusula WHERE

En el ejemplo anterior se mostraba una sencilla consulta de igualdad. La API de SQL también admite varios [expresiones escalares](#scalar-expressions). Las que más se suelen usar son binarias y unarias. Las referencias de propiedad del objeto JSON de origen también son expresiones válidas.

Puede usar los operadores binarios admitidos siguientes:  

|**Nombre de operador**  | **Valores** |
|---------|---------|
|Aritméticos | +,-,*,/,% |
|Bit a bit    | \|, &, ^, <<, >>, >>> (desplazamiento a la derecha con relleno de ceros) |
|Lógicos    | AND, OR, NOT      |
|Comparación | =, !=, &lt;, &gt;, &lt;=, &gt;=, <> |
|String     |  \|\| (concatenar) |

Las consultas siguientes utilizan los operadores binarios:

```sql
    SELECT *
    FROM Families.children[0] c
    WHERE c.grade % 2 = 1     -- matching grades == 5, 1

    SELECT *
    FROM Families.children[0] c
    WHERE c.grade ^ 4 = 1    -- matching grades == 5

    SELECT *
    FROM Families.children[0] c
    WHERE c.grade >= 5    -- matching grades == 5
```

También puede usar los operadores unarios +,-, ~ y no en las consultas, tal como se muestra en los ejemplos siguientes:

```sql
    SELECT *
    FROM Families.children[0] c
    WHERE NOT(c.grade = 5)  -- matching grades == 1

    SELECT *
    FROM Families.children[0] c
    WHERE (-c.grade = -5)  -- matching grades == 5
```

También puede utilizar referencias de propiedad en las consultas. Por ejemplo, `SELECT * FROM Families f WHERE f.isRegistered` devuelve el elemento JSON que contiene la propiedad `isRegistered` con un valor igual a `true`. Cualquier otro valor, como `false`, `null`, `Undefined`, `<number>`, `<string>`, `<object>`, o `<array>`, excluye el elemento del resultado. 

### <a name="equality-and-comparison-operators"></a>Operadores de igualdad y de comparación

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

Para los operadores de comparación como `>`, `>=`, `!=`, `<`, y `<=`, comparación entre los tipos o entre dos objetos o matrices genera `Undefined`.  

Si el resultado de la expresión escalar es `Undefined`, el elemento no se incluye en el resultado, porque `Undefined` no es igual a `true`.

### <a name="logical-and-or-and-not-operators"></a>Operadores lógicos (Y, O y NO)

Los operadores lógicos operan en valores booleanos. Las siguientes tablas muestran las tablas de verdad lógicas para estos operadores:

**operator OR**

| OR | True  | False | No definido |
| --- | --- | --- | --- |
| True  |True  |True  |True  |
| False |True  |False |No definido |
| No definido |True  |No definido |No definido |

**operator AND**

| Y | True  | False | No definido |
| --- | --- | --- | --- |
| True  |True  |False |No definido |
| False |False |False |False |
| No definido |No definido |False |No definido |

**operator NOT**

| NO |  |
| --- | --- |
| True  |False |
| False |True  |
| No definido |No definido |

## <a name="between-keyword"></a>Palabra clave BETWEEN

Al igual que en ANSI SQL, puede usar la palabra clave BETWEEN para expresar consultas en intervalos de valores de cadena o numéricos. Por ejemplo, la siguiente consulta devuelve todos los elementos en el que el curso del primer hijo es 1-5, ambos inclusive.

```sql
    SELECT *
    FROM Families.children[0] c
    WHERE c.grade BETWEEN 1 AND 5
```

A diferencia de en ANSI SQL, se puede usar también la cláusula BETWEEN en la cláusula FROM, como se muestra en el ejemplo siguiente.

```sql
    SELECT (c.grade BETWEEN 0 AND 10)
    FROM Families.children[0] c
```

En la API de SQL, a diferencia de ANSI SQL, puede expresar consultas por rangos en Propiedades de tipos mixtos. Por ejemplo, `grade` podría ser un número similar `5` en algunos elementos y una cadena como `grade4` en otros. En estos casos, al igual que en JavaScript, la comparación entre los dos tipos distintos da como resultado `Undefined`, por lo que se omite el elemento.

> [!TIP]
> Para los tiempos de ejecución de consulta, cree una directiva de indexación que usa un tipo de índice de intervalo en las propiedades numéricas o rutas de acceso que filtra la cláusula BETWEEN.

## <a name="in-keyword"></a>Palabra clave IN

Use la palabra clave IN para comprobar si un valor especificado coincide con cualquier valor en una lista. Por ejemplo, la siguiente consulta devuelve todos los elementos de familia donde el `id` es `WakefieldFamily` o `AndersenFamily`.

```sql
    SELECT *
    FROM Families
    WHERE Families.id IN ('AndersenFamily', 'WakefieldFamily')
```

El ejemplo siguiente devuelve todos los elementos que el estado es cualquiera de los valores especificados:

```sql
    SELECT *
    FROM Families
    WHERE Families.address.state IN ("NY", "WA", "CA", "PA", "OH", "OR", "MI", "WI", "MN", "FL")
```

## <a name="-operator"></a>* operador

El operador especial * proyectos todo el elemento tal cual. Al usarse, debe ser el único campo proyectado. Una consulta como `SELECT * FROM Families f` es válido, pero `SELECT VALUE * FROM Families f` y `SELECT *, f.id FROM Families f` no son válidos. El [consultar primero en este artículo](#query-the-json-items) usa el * operador. 

## <a name="-and--operators"></a>? y?? Operadores

Puede usar el ternario (?) y Coalesce operadores (?) para crear expresiones condicionales, como se muestra en la programación de lenguajes como C# y JavaScript. 

¿Puede usar el? operador para construir nuevas propiedades JSON sobre la marcha. Por ejemplo, la siguiente consulta clasifica los niveles de calificación en `elementary` o `other`:

```sql
     SELECT (c.grade < 5)? "elementary": "other" AS gradeLevel
     FROM Families.children[0] c
```

¿También puede anidar las llamadas a la? operador, como se muestra en la siguiente consulta: 

```sql
    SELECT (c.grade < 5)? "elementary": ((c.grade < 9)? "junior": "high") AS gradeLevel
    FROM Families.children[0] c
```

¿Al igual que con otros operadores de consulta, el? operador excluye elementos si faltan las propiedades que se hace referencia o los tipos que se comparan son diferentes.

Use el?? operador para comprobar eficazmente una propiedad en un elemento cuando se consultan con datos semiestructurados o tipos mixtos. Por ejemplo, la consulta siguiente devuelve `lastName` si está presente, o `surname` si `lastName` no está presente.

```sql
    SELECT f.lastName ?? f.surname AS familyName
    FROM Families f
```

## <a id="TopKeyword"></a>Operador TOP

La palabra clave TOP devuelve el primer `N` número de resultados de la consulta en un orden indefinido. Como práctica recomendada, usar TOP con la cláusula ORDER BY para limitar los resultados de la primera `N` número de los valores ordenados. Combinación de estas dos cláusulas es la única manera previsible de indicar qué filas afecta TOP.

Puede usar la parte superior con un valor constante, como se muestra en el ejemplo siguiente, o con un valor variable usando consultas con parámetros. Para obtener más información, consulte el [consultas parametrizadas](#parameterized-queries) sección.

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

## <a id="OrderByClause"></a>Cláusula ORDER BY

Al igual que en ANSI SQL, puede incluir una cláusula ORDER BY opcional en las consultas. El argumento ASC o DESC opcional especifica si se debe recuperar los resultados en orden ascendente o descendente. ASC es el valor predeterminado.

Por ejemplo, aquí es una consulta que recupera las familias en orden ascendente de nombre de la ciudad de residencia:

```sql
    SELECT f.id, f.address.city
    FROM Families f
    ORDER BY f.address.city
```

Los resultados son:

```json
    [
      {
        "id": "WakefieldFamily",
        "city": "NY"
      },
      {
        "id": "AndersenFamily",
        "city": "Seattle"
      }
    ]
```

La consulta siguiente recupera la familia `id`s en el orden de su fecha de creación del elemento. Elemento `creationDate` es un número que representa el *tiempo de época*, o el tiempo transcurrido desde el 1 de enero de 1970 en segundos.

```sql
    SELECT f.id, f.creationDate
    FROM Families f
    ORDER BY f.creationDate DESC
```

Los resultados son:

```json
    [
      {
        "id": "WakefieldFamily",
        "creationDate": 1431620462
      },
      {
        "id": "AndersenFamily",
        "creationDate": 1431620472
      }
    ]
```

Además, puede ordenar por varias propiedades. Una consulta que ordena por varias propiedades requiere una [índice compuesto](index-policy.md#composite-indexes). Considere la siguiente consulta:

```sql
    SELECT f.id, f.creationDate
    FROM Families f
    ORDER BY f.address.city ASC, f.creationDate DESC
```

Esta consulta recupera la familia `id` en orden ascendente de nombre de la ciudad. Si varios elementos tienen el mismo nombre de ciudad, la consulta se ordena por el `creationDate` en orden descendente.

## <a id="OffsetLimitClause"></a>Cláusula de límite de desplazamiento

LÍMITE de desplazamiento es una cláusula opcional para omitir y llevamos a cabo un número de valores de la consulta. El recuento de desplazamiento y el recuento de límite son necesarios en la cláusula de LIMIT de desplazamiento.

Cuando se usa el límite de desplazamiento junto con una cláusula ORDER BY, el conjunto de resultados se genera haciendo skip y toman los valores ordenados. Si no se usa la cláusula ORDER BY, se producirá un error en un orden determinista de valores.

Por ejemplo, aquí es una consulta que omite el primer valor y devuelve el segundo valor (en orden del nombre de la ciudad de residencia):

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

Esta es una consulta que omite el primer valor y devuelve el segundo valor (sin orden):

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




## <a name="scalar-expressions"></a>Expresiones escalares

La cláusula SELECT es compatible con expresiones escalares como constantes, expresiones aritméticas y expresiones lógicas. La consulta siguiente utiliza una expresión escalar:


```sql
    SELECT ((2 + 11 % 7)-2)/3
```

Los resultados son:

```json
    [{
      "$1": 1.33333
    }]
```

En la siguiente consulta, el resultado de la expresión escalar es un valor booleano:


```sql
    SELECT f.address.city = f.address.state AS AreFromSameCityState
    FROM Families f
```

Los resultados son:

```json
    [
      {
        "AreFromSameCityState": false
      },
      {
        "AreFromSameCityState": true
      }
    ]
```

## <a name="object-and-array-creation"></a>Creación de objetos y matrices

Una característica clave de la API de SQL es la creación de matriz y objeto. El ejemplo anterior crea un nuevo objeto JSON, `AreFromSameCityState`. También puede construir matrices, como se muestra en el ejemplo siguiente:


```sql
    SELECT [f.address.city, f.address.state] AS CityState
    FROM Families f
```

Los resultados son:

```json
    [
      {
        "CityState": [
          "Seattle",
          "WA"
        ]
      },
      {
        "CityState": [
          "NY", 
          "NY"
        ]
      }
    ]
```

La siguiente consulta SQL es otro ejemplo del uso de la matriz dentro de las subconsultas. Esta consulta se obtiene todos los nombres proporcionados distintos de los elementos secundarios en una matriz.

```sql
SELECT f.id, ARRAY(SELECT DISTINCT VALUE c.givenName FROM c IN f.children) as ChildNames
FROM f
```


## <a id="Iteration"></a>Iteración

La API de SQL proporciona compatibilidad para recorrer en iteración matrices JSON, con una nueva construcción de agregado mediante la palabra clave IN en el origen de FROM. En el ejemplo siguiente:

```sql
    SELECT *
    FROM Families.children
```

Los resultados son:

```json
    [
      [
        {
          "firstName": "Henriette Thaulow",
          "gender": "female",
          "grade": 5,
          "pets": [{ "givenName": "Fluffy"}]
        }
      ], 
      [
        {
            "familyName": "Merriam",
            "givenName": "Jesse",
            "gender": "female",
            "grade": 1
        }, 
        {
            "familyName": "Miller",
            "givenName": "Lisa",
            "gender": "female",
            "grade": 8
        }
      ]
    ]
```

La siguiente consulta realiza la iteración en `children` en el `Families` contenedor. La matriz de salida es diferente de la consulta anterior. Este ejemplo se divide `children`y reduce los resultados en una sola matriz:  

```sql
    SELECT *
    FROM c IN Families.children
```

Los resultados son:

```json
    [
      {
          "firstName": "Henriette Thaulow",
          "gender": "female",
          "grade": 5,
          "pets": [{ "givenName": "Fluffy" }]
      },
      {
          "familyName": "Merriam",
          "givenName": "Jesse",
          "gender": "female",
          "grade": 1
      },
      {
          "familyName": "Miller",
          "givenName": "Lisa",
          "gender": "female",
          "grade": 8
      }
    ]
```

Puede filtrar aún más por cada entrada individual de la matriz, como se muestra en el ejemplo siguiente:

```sql
    SELECT c.givenName
    FROM c IN Families.children
    WHERE c.grade = 8
```

Los resultados son:

```json
    [{
      "givenName": "Lisa"
    }]
```

También puede agregar el resultado de una iteración de matriz. Por ejemplo, la consulta siguiente cuenta el número de hijos entre todas las familias de:

```sql
    SELECT COUNT(child)
    FROM child IN Families.children
```

Los resultados son:

```json
    [
      {
        "$1": 3
      }
    ]
```

## <a id="Joins"></a>Combinaciones

En una base de datos relacional, las combinaciones entre tablas son la consecuencia lógica de diseñar esquemas normalizados. En cambio, la API de SQL utiliza el modelo de datos desnormalizado de elementos de esquemas, que es la lógica equivalente de un *autocombinación*.

El lenguaje admite la sintaxis `<from_source1> JOIN <from_source2> JOIN ... JOIN <from_sourceN>`. Esta consulta devuelve un conjunto de tuplas con `N` valores. Cada tupla tiene valores generados mediante la iteración de todos los alias de contenedor en los conjuntos correspondientes. En otras palabras, esta consulta realiza un cruce de productos completo de los conjuntos que participan en la combinación.

En los ejemplos siguientes se muestra cómo funciona la cláusula JOIN. En el ejemplo siguiente, el resultado está vacío, porque el producto cruzado de cada elemento de origen y un conjunto vacío está vacío:

```sql
    SELECT f.id
    FROM Families f
    JOIN f.NonExistent
```

El resultado es el siguiente:

```json
    [{
    }]
```

En el ejemplo siguiente, la combinación es un producto cruzado entre dos objetos JSON, la raíz del elemento `id` y `children` subraíz. El hecho de que `children` es una matriz no es efectiva en la combinación, porque se trata de una sola raíz que es el `children` matriz. El resultado contiene sólo dos resultados, dado que el producto cruzado de todos los elementos en la matriz produce exactamente solo un elemento.

```sql
    SELECT f.id
    FROM Families f
    JOIN f.children
```

Los resultados son:

```json
    [
      {
        "id": "AndersenFamily"
      },
      {
        "id": "WakefieldFamily"
      }
    ]
```

En el ejemplo siguiente se muestra una combinación más convencional:

```sql
    SELECT f.id
    FROM Families f
    JOIN c IN f.children
```

Los resultados son:

```json
    [
      {
        "id": "AndersenFamily"
      },
      {
        "id": "WakefieldFamily"
      },
      {
        "id": "WakefieldFamily"
      }
    ]
```

El origen FROM de la cláusula de combinación es un iterador. Por lo tanto, el flujo en el ejemplo anterior es:  

1. Expanda cada elemento secundario `c` en la matriz.
2. Aplique un producto cruzado con la raíz del elemento `f` con cada elemento secundario `c` que el primer paso simplificada.
3. Por último, el objeto raíz del proyecto `f` `id` propiedad por sí solo.

El primer elemento, `AndersenFamily`, contiene sólo un `children` elemento, por lo que el conjunto de resultados contiene un único objeto. El segundo elemento, `WakefieldFamily`, contiene dos `children`, por lo que el producto cruzado produce dos objetos, uno para cada `children` elemento. Los campos de raíz de ambos elementos son los mismos, tal como se esperaría en un cruce de productos.

Formación de tuplas es la verdadera utilidad de la cláusula de combinación desde el producto cruzado de una forma que en caso contrario, es difícil de proyecto. El ejemplo siguiente los filtros en la combinación de una tupla que permite al usuario seleccionar una condición que cumplen con las tuplas en general.

```sql
    SELECT 
        f.id AS familyName,
        c.givenName AS childGivenName,
        c.firstName AS childFirstName,
        p.givenName AS petName
    FROM Families f
    JOIN c IN f.children
    JOIN p IN c.pets
```

Los resultados son:

```json
    [
      {
        "familyName": "AndersenFamily",
        "childFirstName": "Henriette Thaulow",
        "petName": "Fluffy"
      },
      {
        "familyName": "WakefieldFamily",
        "childGivenName": "Jesse",
        "petName": "Goofy"
      }, 
      {
       "familyName": "WakefieldFamily",
       "childGivenName": "Jesse",
       "petName": "Shadow"
      }
    ]
```

La siguiente extensión del ejemplo anterior realiza una combinación doble. Puede ver el producto cruzado como el pseudocódigo siguiente:

```
    for-each(Family f in Families)
    {
        for-each(Child c in f.children)
        {
            for-each(Pet p in c.pets)
            {
                return (Tuple(f.id AS familyName,
                  c.givenName AS childGivenName,
                  c.firstName AS childFirstName,
                  p.givenName AS petName));
            }
        }
    }
```

`AndersenFamily` tiene un hijo que tiene una mascota, por lo que el producto cruzado produce una fila (1\*1\*1) de esta familia. `WakefieldFamily` tiene dos elementos secundarios, sólo uno de los cuales tiene mascotas, pero ese elemento secundario tiene dos mascotas. Para esta familia de producto cruzado produce 1\*1\*2 = 2 filas.

En el ejemplo siguiente, hay un filtro adicional en `pet`, que excluye todas las tuplas donde el nombre de mascota no es `Shadow`. Puede crear tuplas a partir de matrices, filtrar por cualquiera de los elementos de la tupla y cualquier combinación de los elementos de proyecto.

```sql
    SELECT 
        f.id AS familyName,
        c.givenName AS childGivenName,
        c.firstName AS childFirstName,
        p.givenName AS petName
    FROM Families f
    JOIN c IN f.children
    JOIN p IN c.pets
    WHERE p.givenName = "Shadow"
```

Los resultados son:

```json
    [
      {
       "familyName": "WakefieldFamily",
       "childGivenName": "Jesse",
       "petName": "Shadow"
      }
    ]
```

## <a id="UserDefinedFunctions"></a>Funciones definidas por el usuario (UDF)

La API de SQL proporciona compatibilidad con funciones definidas por el usuario (UDF). Con UDF escalares, puede pasar cero o muchos argumentos y devuelven un resultado único argumento. La API comprueba cada argumento por ser los valores JSON legales.  

La API amplía la sintaxis SQL para admitir el uso de UDF de lógica de aplicación personalizada. Puede registrar UDF con la API de SQL y hacer referencia a ellos en las consultas SQL. De hecho, dichas funciones están exquisitamente diseñadas para que se las pueda llamar desde consultas. Como resultado, las UDF no tienen acceso al objeto de contexto, como otros tipos de JavaScript, como procedimientos almacenados y desencadenadores. Las consultas son de solo lectura y pueden ejecutarse en réplicas principales o secundarias. UDF, a diferencia de otros tipos de JavaScript, están diseñadas para ejecutarse en réplicas secundarias.

El ejemplo siguiente registra una UDF en un contenedor de elementos en la base de datos de Cosmos DB. El ejemplo crea una UDF cuyo nombre es `REGEX_MATCH`. Acepta dos valores de cadena JSON, `input` y `pattern`, y comprueba si el primero coincide con el patrón especificado en el segundo uso de JavaScript `string.match()` función.

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

Ahora, puede usar esta UDF en una proyección de consultas. Las UDF se deben calificar con el prefijo distingue mayúsculas de minúsculas `udf.` al llamar a ellas desde dentro de las consultas.

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

Puede usar la UDF calificada con el `udf.` prefijo dentro de un filtro, como se muestra en el ejemplo siguiente:

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

En esencia, las UDF son expresiones escalares válidas que puede usar en las proyecciones y filtros.

Para ampliar la eficacia de las UDF, mire otro ejemplo con lógica condicional:

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

El ejemplo siguiente ejerce la UDF:

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

Si conocen las propiedades mediante la UDF parámetros no están disponibles en el valor JSON, el parámetro se considera como no definido y la invocación de UDF se omite. De forma similar, si el resultado de la UDF es undefined, no se incluye en el resultado.

Como se muestran en los ejemplos anteriores, las UDF integran el poder del lenguaje JavaScript con la API de SQL. UDF proporcionan una interfaz programable enriquecida para hacer lógica condicional, procedimientos compleja con la Ayuda de capacidades de tiempo de ejecución de JavaScript integradas. La API de SQL proporciona los argumentos a las UDF para cada elemento de origen en el actual WHERE o SELECT fase de procesamiento. El resultado se incorpora a la perfección en la canalización de ejecución general. En resumen, las UDF son excelentes herramientas para hacer la lógica de negocios compleja como parte de las consultas.

## <a id="Aggregates"></a>Funciones de agregado

Las funciones de agregado realizan un cálculo en un conjunto de valores en la cláusula SELECT y devuelven un valor único. Por ejemplo, la consulta siguiente devuelve el recuento de elementos dentro de la `Families` contenedor:

```sql
    SELECT COUNT(1)
    FROM Families f
```

Los resultados son:

```json
    [{
        "$1": 2
    }]
```

También puede devolver solo el valor escalar del agregado mediante el uso de la palabra clave VALUE. Por ejemplo, la siguiente consulta devuelve el número de valores como un único número:

```sql
    SELECT VALUE COUNT(1)
    FROM Families f
```

Los resultados son:

```json
    [ 2 ]
```

También puede combinar las agregaciones con filtros. Por ejemplo, la consulta siguiente devuelve el número de elementos con el estado de la dirección de `WA`.

```sql
    SELECT VALUE COUNT(1)
    FROM Families f
    WHERE f.address.state = "WA"
```

Los resultados son:

```json
    [ 1 ]
```

SQL API admite las siguientes funciones de agregado. SUM y AVG operan en valores numéricos, y trabajan COUNT, MIN y MAX en valores nulos, valores booleanos, cadenas y números.

| Función | DESCRIPCIÓN |
|-------|-------------|
| RECUENTO | Devuelve el número de elementos de la expresión. |
| SUM   | Devuelve la suma de todos los valores de la expresión. |
| MÍN.   | Devuelve el valor mínimo de la expresión. |
| MÁX   | Devuelve el valor máximo de la expresión. |
| AVG   | Devuelve la media de los valores de la expresión. |

También puede agregar a través de los resultados de una iteración de matriz. Para obtener más información, consulte el [iteración](#Iteration) sección.

> [!NOTE]
> En el Explorador de datos del portal de Azure, las consultas de agregación pueden agrega resultados parciales a lo largo de la consulta solo una página. El SDK genera un único valor acumulado en todas las páginas. Para realizar consultas de agregación mediante código, se necesita .NET SDK 1.12.0, .NET Core SDK 1.1.0 o Java SDK 1.9.5 o superior.
>

## <a id="BuiltinFunctions"></a>Funciones integradas

COSMOS DB también admite un número de funciones integradas para operaciones comunes, que se pueden usar dentro de las consultas como funciones definidas por el usuario (UDF).

| Grupo de funciones | Operaciones |
|---------|----------|
| Funciones matemáticas | ABS, CEILING, EXP, FLOOR, LOG, LOG10, POWER, ROUND, SIGN, SQRT, SQUARE, TRUNC, ACOS, ASIN, ATAN, ATN2, COS, COT, DEGREES, PI, RADIANS, SIN y TAN |
| Funciones de comprobación de tipos | IS_ARRAY, IS_BOOL, IS_NULL, IS_NUMBER, IS_OBJECT, IS_STRING, IS_DEFINED e IS_PRIMITIVE |
| Funciones de cadena | CONCAT, CONTAINS, ENDSWITH, INDEX_OF, LEFT, LENGTH, LOWER, LTRIM, REPLACE, REPLICATE, REVERSE, RIGHT, RTRIM, STARTSWITH, SUBSTRING y UPPER |
| Funciones de matriz | ARRAY_CONCAT, ARRAY_CONTAINS, ARRAY_LENGTH y ARRAY_SLICE |
| Funciones espaciales | ST_DISTANCE, ST_WITHIN, ST_INTERSECTS, ST_ISVALID y ST_ISVALIDDETAILED |

Si actualmente usa una función definida por el usuario (UDF) para el que una función integrada ahora está disponible, la función integrada correspondiente será más eficaz y a ejecutar más rápidamente.

La principal diferencia entre funciones de Cosmos DB y SQL ANSI es que las funciones de Cosmos DB están diseñadas para funcionar bien con datos sin esquema y el esquema mixto. Por ejemplo, si una propiedad falta o tiene un valor no numérico como `unknown`, se omite el elemento en lugar de devolver un error.

### <a name="mathematical-functions"></a>Funciones matemáticas

Las funciones matemáticas realizan un cálculo, basado en valores de entrada proporcionados como argumentos, y devuelven un valor numérico. Esta es una tabla de las funciones matemáticas integradas admitidas.

| Uso | DESCRIPCIÓN |
|----------|--------|
| ABS (num_expr) | Devuelve el valor absoluto (positivo) de la expresión numérica especificada. |
| CEILING (num_expr) | Devuelve el valor entero más pequeño mayor o igual que la expresión numérica especificada. |
| FLOOR (num_expr) | Devuelve el valor entero más grande menor o igual que la expresión numérica especificada. |
| EXP (num_expr) | Devuelve el exponente de la expresión numérica especificada. |
| REGISTRO (num_expr, base) | Devuelve el logaritmo natural de la expresión numérica especificada, o bien el logaritmo que utiliza la base especificada. |
| LOG10 (num_expr) | Devuelve el valor logarítmico de base 10 de la expresión numérica especificada. |
| ROUND (num_expr) | Devuelve un valor numérico, redondeado al valor entero más cercano. |
| TRUNC (num_expr) | Devuelve un valor numérico, truncado al valor entero más cercano. |
| SQRT (num_expr) | Devuelve la raíz cuadrada de la expresión numérica especificada. |
| SQUARE (num_expr) | Devuelve el cuadrado de la expresión numérica especificada. |
| POWER (num_expr, num_expr) | Devuelve la potencia de la expresión numérica especificada al valor especificado. |
| SIGN (num_expr) | Devuelve el valor de signo (-1, 0, 1) de la expresión numérica especificada. |
| ACOS (num_expr) | Devuelve el ángulo, en radianes, cuyo coseno es la expresión numérica especificada; también se denomina arcocoseno. |
| ASIN (num_expr) | Devuelve el ángulo, en radianes, cuyo seno es la expresión numérica especificada. Esta función también se denomina arcoseno. |
| ATAN (num_expr) | Devuelve el ángulo, en radianes, cuya tangente es la expresión numérica especificada. Esta función también se denomina arcotangente. |
| ATN2 (num_expr) | Devuelve el ángulo, en radianes, entre el eje x positivo y el rayo desde el origen hasta el punto (y, x), donde x e y son los valores de las dos expresiones de punto flotante especificadas. |
| COS (num_expr) | Devuelve el coseno trigonométrico del ángulo especificado, en radianes, en la expresión especificada. |
| COT (num_expr) | Devuelve la cotangente trigonométrica del ángulo especificado, en radianes, en la expresión numérica especificada. |
| DEGREES (num_expr) | Devuelve el ángulo correspondiente en grados de un ángulo especificado en radianes. |
| PI () | Devuelve el valor constante de PI. |
| RADIANS (num_expr) | Devuelve radianes cuando se especifica una expresión numérica en grados. |
| SIN (num_expr) | Devuelve el seno trigonométrico del ángulo especificado, en radianes, en la expresión especificada. |
| TAN (num_expr) | Devuelve la tangente de la expresión de entrada en la expresión especificada. |

Puede ejecutar consultas similar al ejemplo siguiente:

```sql
    SELECT VALUE ABS(-4)
```

El resultado es el siguiente:

```json
    [4]
```

### <a name="type-checking-functions"></a>Funciones de comprobación de tipos

Las funciones de comprobación de tipos permiten comprobar el tipo de una expresión dentro de una consulta SQL. Puede usar las funciones de comprobación de tipos para determinar los tipos de propiedades de artículos sobre la marcha, cuando están variable o desconocido. Esta es una tabla de funciones de comprobación de tipos integradas admitidas:

| **Uso** | **Descripción** |
|-----------|------------|
| [IS_ARRAY (expr)](sql-api-query-reference.md#bk_is_array) | Devuelve un valor booleano que indica si el tipo del valor es una matriz. |
| [IS_BOOL (expr)](sql-api-query-reference.md#bk_is_bool) | Devuelve un valor booleano que indica si el tipo del valor es un valor booleano. |
| [IS_NULL (expr)](sql-api-query-reference.md#bk_is_null) | Devuelve un valor booleano que indica si el tipo del valor es nulo. |
| [IS_NUMBER (expr)](sql-api-query-reference.md#bk_is_number) | Devuelve un valor booleano que indica si el tipo del valor es un número. |
| [IS_OBJECT (expr)](sql-api-query-reference.md#bk_is_object) | Devuelve un valor booleano que indica si el tipo del valor es un objeto JSON. |
| [IS_STRING (expr)](sql-api-query-reference.md#bk_is_string) | Devuelve un valor booleano que indica si el tipo del valor es una cadena. |
| [IS_DEFINED (expr)](sql-api-query-reference.md#bk_is_defined) | Devuelve un valor booleano que indica si se ha asignado un valor a la propiedad. |
| [IS_PRIMITIVE (expr)](sql-api-query-reference.md#bk_is_primitive) | Devuelve un valor booleano que indica si el tipo del valor es una cadena, número, booleano o null. |

Uso de estas funciones, puede ejecutar consultas similar al ejemplo siguiente:

```sql
    SELECT VALUE IS_NUMBER(-4)
```

El resultado es el siguiente:

```json
    [true]
```

### <a name="string-functions"></a>Funciones de cadena

Las siguientes funciones escalares realizan una operación en un valor de cadena de entrada y devuelven un valor de cadena, numérico o booleano. A continuación se facilita una tabla de funciones de cadena integradas:

| Uso | DESCRIPCIÓN |
| --- | --- |
| [LENGTH (str_expr)](sql-api-query-reference.md#bk_length) | Devuelve el número de caracteres de la expresión de cadena especificada. |
| [CONCAT (str_expr, str_expr [, str_expr])](sql-api-query-reference.md#bk_concat) | Devuelve una cadena que es el resultado de concatenar dos o más valores de cadena. |
| [SUBSTRING (str_expr, num_expr, num_expr)](sql-api-query-reference.md#bk_substring) | Devuelve parte de una expresión de cadena. |
| [STARTSWITH (str_expr, str_expr)](sql-api-query-reference.md#bk_startswith) | Devuelve un valor booleano que indica si la primera expresión de cadena empieza con la segunda. |
| [ENDSWITH (str_expr, str_expr)](sql-api-query-reference.md#bk_endswith) | Devuelve un valor booleano que indica si la primera expresión de cadena finaliza con la segunda. |
| [CONTAINS (str_expr, str_expr)](sql-api-query-reference.md#bk_contains) | Devuelve un valor booleano que indica si la primera expresión de cadena contiene la segunda. |
| [INDEX_OF (str_expr, str_expr)](sql-api-query-reference.md#bk_index_of) | Devuelve la posición inicial de la primera aparición de la segunda cadena de expresión dentro de la primera expresión de cadena especificada, o -1 si no se encuentra la cadena. |
| [LEFT (str_expr, num_expr)](sql-api-query-reference.md#bk_left) | Devuelve la parte izquierda de una cadena con el número especificado de caracteres. |
| [RIGHT (str_expr, num_expr)](sql-api-query-reference.md#bk_right) | Devuelve la parte derecha de una cadena con el número especificado de caracteres. |
| [LTRIM (str_expr)](sql-api-query-reference.md#bk_ltrim) | Devuelve una expresión de cadena después de quitar los espacios en blanco iniciales. |
| [RTRIM (str_expr)](sql-api-query-reference.md#bk_rtrim) | Devuelve una expresión de cadena después de truncar todos los espacios en blanco finales. |
| [LOWER (str_expr)](sql-api-query-reference.md#bk_lower) | Devuelve una expresión de cadena después de convertir los datos de caracteres en mayúsculas a minúsculas. |
| [UPPER (str_expr)](sql-api-query-reference.md#bk_upper) | Devuelve una expresión de cadena después de convertir datos de caracteres en minúsculas a mayúsculas. |
| [REPLACE (str_expr, str_expr, str_expr)](sql-api-query-reference.md#bk_replace) | Reemplaza todas las apariciones de un valor de cadena especificado por otro valor de cadena. |
| [REPLICATE (str_expr, num_expr)](sql-api-query-reference.md#bk_replicate) | Repite un valor de cadena un número especificado de veces. |
| [REVERSE (str_expr)](sql-api-query-reference.md#bk_reverse) | Devuelve el orden inverso de un valor de cadena. |

Con estas funciones, puede ejecutar consultas similares a lo siguiente, que devuelve la familia `id` en mayúsculas:

```sql
    SELECT VALUE UPPER(Families.id)
    FROM Families
```

Los resultados son:

```json
    [
        "WAKEFIELDFAMILY",
        "ANDERSENFAMILY"
    ]
```

O concatenar cadenas, como en este ejemplo:

```sql
    SELECT Families.id, CONCAT(Families.address.city, ",", Families.address.state) AS location
    FROM Families
```

Los resultados son:

```json
    [{
      "id": "WakefieldFamily",
      "location": "NY,NY"
    },
    {
      "id": "AndersenFamily",
      "location": "Seattle,WA"
    }]
```

También puede usar las funciones de cadena en la cláusula WHERE para filtrar los resultados, como en el ejemplo siguiente:

```sql
    SELECT Families.id, Families.address.city
    FROM Families
    WHERE STARTSWITH(Families.id, "Wakefield")
```

Los resultados son:

```json
    [{
      "id": "WakefieldFamily",
      "city": "NY"
    }]
```

### <a name="array-functions"></a>Funciones de matriz

Las siguientes funciones escalares realizan una operación en un valor de entrada de matriz y devuelvan un valor numérico, booleano o valor de matriz. A continuación se facilita una tabla de funciones de matriz integradas:

| Uso | DESCRIPCIÓN |
| --- | --- |
| [ARRAY_LENGTH (arr_expr)](sql-api-query-reference.md#bk_array_length) |Devuelve el número de elementos de la expresión de matriz especificada. |
| [ARRAY_CONCAT (arr_expr, arr_expr [, arr_expr])](sql-api-query-reference.md#bk_array_concat) |Devuelve una matriz que es el resultado de concatenar dos o más valores de la matriz. |
| [ARRAY_CONTAINS (arr_expr, expr [, bool_expr])](sql-api-query-reference.md#bk_array_contains) |Devuelve un valor booleano que indica si la matriz contiene el valor especificado. Puede especificar si la coincidencia es completa o parcial. |
| [ARRAY_SLICE (arr_expr, num_expr [, num_expr])](sql-api-query-reference.md#bk_array_slice) |Devuelve parte de una expresión de matriz. |

Usar funciones de matriz para manipular matrices en JSON. Por ejemplo, aquí es una consulta que devuelve todos los elemento `id`s en el que uno de los `parents` es `Robin Wakefield`: 

```sql
    SELECT Families.id 
    FROM Families 
    WHERE ARRAY_CONTAINS(Families.parents, { givenName: "Robin", familyName: "Wakefield" })
```

El resultado es el siguiente:

```json
    [{
      "id": "WakefieldFamily"
    }]
```

Puede especificar un fragmento parcial para los elementos que coinciden en la matriz. La siguiente consulta busca todos los elemento `id`que tienen `parents` con el `givenName` de `Robin`:

```sql
    SELECT Families.id 
    FROM Families 
    WHERE ARRAY_CONTAINS(Families.parents, { givenName: "Robin" }, true)
```

El resultado es el siguiente:

```json
    [{
      "id": "WakefieldFamily"
    }]
```

Este es otro ejemplo que usa ARRAY_LENGTH para obtener el número de `children` por familia:

```sql
    SELECT Families.id, ARRAY_LENGTH(Families.children) AS numberOfChildren
    FROM Families 
```

Los resultados son:

```json
    [{
      "id": "WakefieldFamily",
      "numberOfChildren": 2
    },
    {
      "id": "AndersenFamily",
      "numberOfChildren": 1
    }]
```

### <a name="spatial-functions"></a>Funciones espaciales

COSMOS DB admite las siguientes funciones integradas de Open Geospatial Consortium (OGC) para realizar consultas geoespaciales: 

| Uso | DESCRIPCIÓN |
| --- | --- |
| ST_DISTANCE (point_expr, point_expr) | Devuelve la distancia entre el objeto de GeoJSON dos `Point`, `Polygon`, o `LineString` expresiones. |
| T_WITHIN (point_expr, polygon_expr) | Devuelve una expresión booleana que indica si el primer objeto de GeoJSON (`Point`, `Polygon`, o `LineString`) está dentro del segundo objeto de GeoJSON (`Point`, `Polygon`, o `LineString`). |
| ST_INTERSECTS (spatial_expr, spatial_expr) | Devuelve una expresión booleana que indica si los dos especificados de objetos de GeoJSON (`Point`, `Polygon`, o `LineString`) forman una intersección. |
| ST_ISVALID | Devuelve un valor booleano que indica si el objeto de GeoJSON especificado `Point`, `Polygon`, o `LineString` expresión es válida. |
| ST_ISVALIDDETAILED | Devuelve un valor JSON que contiene un valor booleano si el objeto de GeoJSON especificado `Point`, `Polygon`, o `LineString` expresión es válida y si no es válido, el motivo como un valor de cadena. |

Puede usar funciones espaciales para realizar consultas de proximidad con datos espaciales. Por ejemplo, aquí es una consulta que devuelve todos los elementos de familia que están dentro de 30 km de una ubicación especificada mediante la función integrada ST_DISTANCE:

```sql
    SELECT f.id
    FROM Families f
    WHERE ST_DISTANCE(f.location, {'type': 'Point', 'coordinates':[31.9, -4.8]}) < 30000
```

El resultado es el siguiente:

```json
    [{
      "id": "WakefieldFamily"
    }]
```

Para más información acerca de la compatibilidad geoespacial en Cosmos DB, consulte [Uso de datos geoespaciales en Azure Cosmos DB](geospatial.md), 

## <a name="parameterized-queries"></a>Consultas parametrizadas

COSMOS DB admite consultas con parámetros que se expresa mediante la notación @ ya conocida. Proporciona un control sólido y secuencias de escape de entrada de usuario SQL con parámetros y evita la exposición accidental de datos a través de la inyección de código SQL.

Por ejemplo, puede escribir una consulta que toma `lastName` y `address.state` como parámetros y ejecutarla para distintos valores de `lastName` y `address.state` basándose en la entrada del usuario.

```sql
    SELECT *
    FROM Families f
    WHERE f.lastName = @lastName AND f.address.state = @addressState
```

A continuación, puede enviar esta solicitud a Cosmos DB como una consulta parametrizada de JSON similar al siguiente:

```sql
    {
        "query": "SELECT * FROM Families f WHERE f.lastName = @lastName AND f.address.state = @addressState",
        "parameters": [
            {"name": "@lastName", "value": "Wakefield"},
            {"name": "@addressState", "value": "NY"},
        ]
    }
```

El ejemplo siguiente establece el argumento superior con una consulta parametrizada: 

```sql
    {
        "query": "SELECT TOP @n * FROM Families",
        "parameters": [
            {"name": "@n", "value": 10},
        ]
    }
```

Los valores de parámetro pueden ser cualquier valor JSON válido: cadenas, números, booleanos, null, o incluso matrices o JSON anidado. Puesto que Cosmos DB es sin esquema, los parámetros no se validan frente a cualquier tipo.

## <a id="JavaScriptIntegration"></a>Integración de JavaScript

Azure Cosmos DB proporciona un modelo de programación para ejecutar lógica de aplicaciones basadas en JavaScript directamente en contenedores, mediante procedimientos almacenados y desencadenadores. Este modelo es compatible con:

* Operaciones CRUD transaccionales de alto rendimiento y consultas con respecto a elementos en un contenedor, en virtud de la integración profunda de runtime de JavaScript en el motor de base de datos.
* Un modelo natural de flujo de control, variables de ámbito y asignación e integración de primitivos de control de excepciones con transacciones de base de datos. 

Para obtener más información acerca de la integración de JavaScript de Azure Cosmos DB, consulte el [API de JavaScript del lado servidor](#JavaScriptServerSideApi) sección.

### <a name="operator-evaluation"></a>Evaluación de operadores

COSMOS DB, en virtud de ser una base de datos JSON, Establece paralelismos con los operadores de JavaScript y semántica de evaluación. COSMOS DB intenta conservar la semántica de JavaScript en términos de compatibilidad de JSON, pero la evaluación de operaciones se desvía en algunas instancias.

En la API de SQL, a diferencia del lenguaje SQL tradicional, los tipos de valores no suelen conocerse hasta que la API de recupera los valores de la base de datos. Para ejecutar consultas de forma eficaz, la mayoría de los operadores tienen requisitos de tipo estrictos.

A diferencia de JavaScript, la API de SQL no realiza conversiones implícitas. Por ejemplo, una consulta como `SELECT * FROM Person p WHERE p.Age = 21` coincide con los elementos que contienen un `Age` propiedad cuyo valor es `21`. No coincide con cualquier otro elemento cuya propiedad `Age` propiedad coincide con variaciones posiblemente infinitas como `twenty-one`, `021`, o `21.0`. Esto contrasta con JavaScript, donde los valores de cadena se convierten implícitamente a los números en función del operador, por ejemplo: `==`. Este comportamiento de SQL API es crucial para la coincidencia de índices eficaz.

## <a id="ExecutingSqlQueries"></a>Ejecución de consultas SQL

Cualquier lenguaje capaz de realizar solicitudes HTTP/HTTPS puede llamar a la API de REST de Cosmos DB. COSMOS DB también ofrece bibliotecas de programación para los lenguajes de programación. NET, Node.js, JavaScript y Python. La API de REST y bibliotecas se admiten consultas a través de SQL y el SDK de .NET también admite [consultas LINQ](#Linq).

En los ejemplos siguientes se muestra cómo crear una consulta y enviarla a una cuenta de la base de datos de Cosmos DB.

### <a id="RestAPI"></a>API DE REST

Cosmos DB ofrece un modelo de programación RESTful sobre HTTP. El modelo de recursos consta de un conjunto de recursos en una cuenta de base de datos, lo que las disposiciones de una suscripción de Azure. La cuenta de base de datos consta de un conjunto de *bases de datos*, cada uno de los cuales puede contener varios *contenedores*, que a su vez contienen *elementos*, UDF y otros tipos de recursos. Cada recurso de Cosmos DB es direccionable mediante un URI lógico y estable. Un conjunto de recursos se denomina un *fuente*. 

El modelo de interacción básico con estos recursos es a través de los verbos HTTP `GET`, `PUT`, `POST`, y `DELETE`, con sus interpretaciones estándares. Use `POST` para crear un nuevo recurso, ejecutar un procedimiento almacenado o emitir una consulta de Cosmos DB. Las consultas siempre son operaciones de solo lectura sin efectos secundarios.

Los ejemplos siguientes muestran un `POST` para una consulta SQL API en los elementos de ejemplo. La consulta tiene un filtro sencillo en el JSON `name` propiedad. El `x-ms-documentdb-isquery` y Content-Type: `application/query+json` encabezados indican que la operación es una consulta. Reemplace `mysqlapicosmosdb.documents.azure.com:443` con el URI para la cuenta de Cosmos DB.

```json
    POST https://mysqlapicosmosdb.documents.azure.com:443/docs HTTP/1.1
    ...
    x-ms-documentdb-isquery: True
    Content-Type: application/query+json

    {
        "query": "SELECT * FROM Families f WHERE f.id = @familyId",
        "parameters": [
            {"name": "@familyId", "value": "AndersenFamily"}
        ]
    }
```

Los resultados son:

```json
    HTTP/1.1 200 Ok
    x-ms-activity-id: 8b4678fa-a947-47d3-8dd3-549a40da6eed
    x-ms-item-count: 1
    x-ms-request-charge: 0.32

    {  
       "_rid":"u1NXANcKogE=",
       "Documents":[  
          {  
             "id":"AndersenFamily",
             "lastName":"Andersen",
             "parents":[  
                {  
                   "firstName":"Thomas"
                },
                {  
                   "firstName":"Mary Kay"
                }
             ],
             "children":[  
                {  
                   "firstName":"Henriette Thaulow",
                   "gender":"female",
                   "grade":5,
                   "pets":[  
                      {  
                         "givenName":"Fluffy"
                      }
                   ]
                }
             ],
             "address":{  
                "state":"WA",
                "county":"King",
                "city":"Seattle"
             },
             "_rid":"u1NXANcKogEcAAAAAAAAAA==",
             "_ts":1407691744,
             "_self":"dbs\/u1NXAA==\/colls\/u1NXANcKogE=\/docs\/u1NXANcKogEcAAAAAAAAAA==\/",
             "_etag":"00002b00-0000-0000-0000-53e7abe00000",
             "_attachments":"_attachments\/"
          }
       ],
       "count":1
    }
```

La siguiente consulta más compleja devuelve varios resultados de una combinación:

```json
    POST https://https://mysqlapicosmosdb.documents.azure.com:443/docs HTTP/1.1
    ...
    x-ms-documentdb-isquery: True
    Content-Type: application/query+json

    {
        "query": "SELECT
                     f.id AS familyName,
                     c.givenName AS childGivenName,
                     c.firstName AS childFirstName,
                     p.givenName AS petName
                  FROM Families f
                  JOIN c IN f.children
                  JOIN p in c.pets",
        "parameters": [] 
    }
```

Los resultados son: 

```json
    HTTP/1.1 200 Ok
    x-ms-activity-id: 568f34e3-5695-44d3-9b7d-62f8b83e509d
    x-ms-item-count: 1
    x-ms-request-charge: 7.84

    {  
       "_rid":"u1NXANcKogE=",
       "Documents":[  
          {  
             "familyName":"AndersenFamily",
             "childFirstName":"Henriette Thaulow",
             "petName":"Fluffy"
          },
          {  
             "familyName":"WakefieldFamily",
             "childGivenName":"Jesse",
             "petName":"Goofy"
          },
          {  
             "familyName":"WakefieldFamily",
             "childGivenName":"Jesse",
             "petName":"Shadow"
          }
       ],
       "count":3
    }
```

Si los resultados de una consulta no caben en una sola página, la API de REST devuelve un token de continuación a través de la `x-ms-continuation-token` encabezado de respuesta. Los clientes pueden paginar los resultados al incluir el encabezado en los resultados posteriores. También puede controlar el número de resultados por página a través de la `x-ms-max-item-count` encabezado de número. 

Si una consulta tiene una función de agregación como recuento, la página de consulta puede devolver un valor agregado parcialmente a través solo una página de resultados. Los clientes deben realizar una agregación de segundo nivel con estos resultados para generar los resultados finales. Por ejemplo, sumar los recuentos devueltos en las páginas individuales para devolver el recuento total.

Para administrar la directiva de coherencia de datos para las consultas, use el `x-ms-consistency-level` encabezado como en todas las solicitudes de API de REST. Coherencia de sesión también requiere la versión más reciente de eco `x-ms-session-token` encabezado de cookie en la solicitud de consulta. La directiva de indexación del contenedor consultado también puede afectar a la coherencia de los resultados de la consulta. Con la configuración de directiva para los contenedores de indexación predeterminada, el índice siempre está actualizado con el contenido del elemento y resultados de la consulta coinciden con la coherencia elegida para los datos. Para obtener más información, consulte [niveles de coherencia de Azure Cosmos DB][consistency-levels].

Si la directiva de indexación configurada en el contenedor no se admite la consulta especificada, el servidor de Azure Cosmos DB devuelve 400 "Bad Request". Este mensaje de error se devuelve para las consultas con excluir explícitamente de la indexación de rutas de acceso. Puede especificar el `x-ms-documentdb-query-enable-scan` encabezado para permitir que la consulta se realiza un examen cuando un índice no está disponible.

Puede obtener métricas detalladas sobre la ejecución de la consulta estableciendo el `x-ms-documentdb-populatequerymetrics` encabezado para `true`. Para más información, vea [SQL query metrics for Azure Cosmos DB](sql-api-query-metrics.md) (Métricas de consulta de SQL para la API de Azure Cosmos DB).

### <a id="DotNetSdk"></a>C#(SDK) DE .NET

El SDK de .NET admite la realización de consultas de LINQ y SQL. El ejemplo siguiente muestra cómo realizar la consulta de filtro anterior con. NET:

```csharp
    foreach (var family in client.CreateDocumentQuery(containerLink,
        "SELECT * FROM Families f WHERE f.id = \"AndersenFamily\""))
    {
        Console.WriteLine("\tRead {0} from SQL", family);
    }

    SqlQuerySpec query = new SqlQuerySpec("SELECT * FROM Families f WHERE f.id = @familyId");
    query.Parameters = new SqlParameterCollection();
    query.Parameters.Add(new SqlParameter("@familyId", "AndersenFamily"));

    foreach (var family in client.CreateDocumentQuery(containerLink, query))
    {
        Console.WriteLine("\tRead {0} from parameterized SQL", family);
    }

    foreach (var family in (
        from f in client.CreateDocumentQuery(containerLink)
        where f.Id == "AndersenFamily"
        select f))
    {
        Console.WriteLine("\tRead {0} from LINQ query", family);
    }

    foreach (var family in client.CreateDocumentQuery(containerLink)
        .Where(f => f.Id == "AndersenFamily")
        .Select(f => f))
    {
        Console.WriteLine("\tRead {0} from LINQ lambda", family);
    }
```

El ejemplo siguiente compara dos propiedades para igualdad dentro de cada elemento y usa proyecciones anónimas.

```csharp
    foreach (var family in client.CreateDocumentQuery(containerLink,
        @"SELECT {""Name"": f.id, ""City"":f.address.city} AS Family
        FROM Families f
        WHERE f.address.city = f.address.state"))
    {
        Console.WriteLine("\tRead {0} from SQL", family);
    }

    foreach (var family in (
        from f in client.CreateDocumentQuery<Family>(containerLink)
        where f.address.city == f.address.state
        select new { Name = f.Id, City = f.address.city }))
    {
        Console.WriteLine("\tRead {0} from LINQ query", family);
    }

    foreach (var family in
        client.CreateDocumentQuery<Family>(containerLink)
        .Where(f => f.address.city == f.address.state)
        .Select(f => new { Name = f.Id, City = f.address.city }))
    {
        Console.WriteLine("\tRead {0} from LINQ lambda", family);
    }
```

El ejemplo siguiente muestra combinaciones, expresadas a través de LINQ `SelectMany`.

```csharp
    foreach (var pet in client.CreateDocumentQuery(containerLink,
          @"SELECT p
            FROM Families f
                 JOIN c IN f.children
                 JOIN p in c.pets
            WHERE p.givenName = ""Shadow"""))
    {
        Console.WriteLine("\tRead {0} from SQL", pet);
    }

    // Equivalent in Lambda expressions:
    foreach (var pet in
        client.CreateDocumentQuery<Family>(containerLink)
        .SelectMany(f => f.children)
        .SelectMany(c => c.pets)
        .Where(p => p.givenName == "Shadow"))
    {
        Console.WriteLine("\tRead {0} from LINQ lambda", pet);
    }
```

El cliente de .NET se itera automáticamente a través de todas las páginas de resultados de la consulta en el `foreach` se bloquea, tal como se muestra en el ejemplo anterior. Las opciones de consulta introdujeron en la [API de REST](#RestAPI) sección también están disponibles en el SDK de .NET mediante el `FeedOptions` y `FeedResponse` clases en el `CreateDocumentQuery` método. Puede controlar el número de páginas mediante el `MaxItemCount` configuración.

Puede controlar expresamente la paginación creando `IDocumentQueryable` mediante el `IQueryable` objeto, a continuación, leyendo el `ResponseContinuationToken` nuevo de valores y devolviéndolos como `RequestContinuationToken` en `FeedOptions`. Puede establecer `EnableScanInQuery` para habilitar los exámenes cuando la consulta no es compatible con la directiva de indexación configurada. Para los contenedores con particiones, puede usar `PartitionKey` para ejecutar la consulta en una sola partición, aunque Azure Cosmos DB puede extraer automáticamente esto a partir del texto de consulta. Puede usar `EnableCrossPartitionQuery` para ejecutar consultas en varias particiones.

Para obtener más ejemplos de .NET con las consultas, vea el [ejemplos de .NET de Azure Cosmos DB](https://github.com/Azure/azure-cosmosdb-dotnet) en GitHub.

### <a id="JavaScriptServerSideApi"></a>API del servidor de JavaScript

COSMOS DB proporciona un modelo de programación para ejecutar lógica de aplicación basada en JavaScript directamente en contenedores, mediante procedimientos almacenados y desencadenadores. La lógica de JavaScript registrada en el nivel de contenedor, a continuación, puede emitir operaciones de base de datos en los elementos del contenedor determinado, encapsulado en transacciones ACID ambientales.

El ejemplo siguiente muestra cómo usar `queryDocuments` en el servidor de JavaScript API para realizar consultas desde dentro de procedimientos almacenados y desencadenadores:

```javascript
    function findName(givenName, familyName) {
        var context = getContext();
        var containerManager = context.getCollection();
        var containerLink = containerManager.getSelfLink()

        // create a new item.
        containerManager.createDocument(containerLink,
            { givenName: givenName, familyName: familyName },
            function (err, documentCreated) {
                if (err) throw new Error(err.message);

                // filter items by familyName
                var filterQuery = "SELECT * from root r WHERE r.familyName = 'Wakefield'";
                containerManager.queryDocuments(containerLink,
                    filterQuery,
                    function (err, matchingDocuments) {
                        if (err) throw new Error(err.message);
    context.getResponse().setBody(matchingDocuments.length);

                        // Replace the familyName for all items that satisfied the query.
                        for (var i = 0; i < matchingDocuments.length; i++) {
                            matchingDocuments[i].familyName = "Robin Wakefield";
                            // we don't need to execute a callback because they are in parallel
                            containerManager.replaceDocument(matchingDocuments[i]._self,
                                matchingDocuments[i]);
                        }
                    })
            });
    }
```

## <a id="Linq"></a>API de LINQ to SQL

LINQ es un modelo de programación de .NET que expresa cálculos como consultas de objeto secuencias. Cosmos DB proporciona una biblioteca del cliente para interactuar con LINQ facilitando una conversión entre objetos JSON y .NET, y una asignación a partir de un subconjunto de consultas de LINQ a consultas de Cosmos DB.

El siguiente diagrama muestra la arquitectura de consultas compatibles con LINQ usa Cosmos DB. Con el cliente de Cosmos DB, puede crear un `IQueryable` objeto que consulta el proveedor de consultas de Cosmos DB directamente y la consulta LINQ se traduce en una consulta de Cosmos DB. A continuación, pase la consulta al servidor de Cosmos DB, que recupera un conjunto de resultados en formato JSON. El deserializador JSON convierte los resultados en una secuencia de objetos .NET en el lado cliente.

![Arquitectura de consultas compatibles con LINQ que usa la API de SQL: sintaxis SQL, lenguaje de consulta JSON, conceptos de base de datos y consultas SQL][1]

### <a name="net-and-json-mapping"></a>Asignación de .NET y JSON

La asignación entre objetos .NET y elementos de JSON es natural. Cada campo de miembro de datos se asigna a un objeto JSON, donde el nombre del campo se asigna a la *clave* asigna parte del objeto y el valor de forma recursiva a la *valor* parte del objeto. El código siguiente asigna el `Family` clase a un elemento JSON y, a continuación, se crea un `Family` objeto:

```csharp
    public class Family
    {
        [JsonProperty(PropertyName="id")]
        public string Id;
        public Parent[] parents;
        public Child[] children;
        public bool isRegistered;
    };

    public struct Parent
    {
        public string familyName;
        public string givenName;
    };

    public class Child
    {
        public string familyName;
        public string givenName;
        public string gender;
        public int grade;
        public List<Pet> pets;
    };

    public class Pet
    {
        public string givenName;
    };

    public class Address
    {
        public string state;
        public string county;
        public string city;
    };

    // Create a Family object.
    Parent mother = new Parent { familyName= "Wakefield", givenName="Robin" };
    Parent father = new Parent { familyName = "Miller", givenName = "Ben" };
    Child child = new Child { familyName="Merriam", givenName="Jesse", gender="female", grade=1 };
    Pet pet = new Pet { givenName = "Fluffy" };
    Address address = new Address { state = "NY", county = "Manhattan", city = "NY" };
    Family family = new Family { Id = "WakefieldFamily", parents = new Parent [] { mother, father}, children = new Child[] { child }, isRegistered = false };
```

El ejemplo anterior crea el elemento JSON siguiente:

```json
    {
        "id": "WakefieldFamily",
        "parents": [
            { "familyName": "Wakefield", "givenName": "Robin" },
            { "familyName": "Miller", "givenName": "Ben" }
        ],
        "children": [
            {
                "familyName": "Merriam",
                "givenName": "Jesse",
                "gender": "female",
                "grade": 1,
                "pets": [
                    { "givenName": "Goofy" },
                    { "givenName": "Shadow" }
                ]
            },
            { 
              "familyName": "Miller",
              "givenName": "Lisa",
              "gender": "female",
              "grade": 8
            }
        ],
        "address": { "state": "NY", "county": "Manhattan", "city": "NY" },
        "isRegistered": false
    };
```

### <a name="linq-to-sql-translation"></a>LINQ para traducción de lenguaje SQL

El proveedor de consulta de Cosmos DB realiza una mejor opción de asignación desde una consulta de LINQ a una consulta SQL de Cosmos DB. La descripción siguiente se da por supuesto un conocimiento básico de LINQ.

El sistema de tipos de proveedor de consulta admite solo los tipos primitivos de JSON: numérico, booleano, cadena y null. 

El proveedor de consultas admite las siguientes expresiones escalares:

- Valores constantes, incluidos los valores constantes de los tipos de datos primitivos durante la evaluación de la consulta.
  
- Expresiones de índice de propiedad o matriz que hacen referencia a la propiedad de un objeto o un elemento de matriz. Por ejemplo:
  
  ```
    family.Id;
    family.children[0].familyName;
    family.children[0].grade;
    family.children[n].grade; //n is an int variable
  ```
  
- Expresiones aritméticas, incluidas expresiones aritméticas comunes en valores numéricos y booleanos. Para obtener una lista completa, consulte el [especificación de SQL de Azure Cosmos DB](https://go.microsoft.com/fwlink/p/?LinkID=510612).
  
  ```
    2 * family.children[0].grade;
    x + y;
  ```
  
- Expresiones de comparación de cadena, que consisten en comparar un valor de cadena a algún valor de cadena constante.  
  
  ```
    mother.familyName == "Wakefield";
    child.givenName == s; //s is a string variable
  ```
  
- Expresiones de creación de objeto o matriz, que devuelven un objeto de tipo de valor compuesto o tipo anónimo o una matriz de estos objetos. Puede anidar estos valores.
  
  ```
    new Parent { familyName = "Wakefield", givenName = "Robin" };
    new { first = 1, second = 2 }; //an anonymous type with two fields  
    new int[] { 3, child.grade, 5 };
  ```

### <a id="SupportedLinqOperators"></a>Operadores LINQ admitidos

El proveedor LINQ incluido con el SDK de .NET de SQL admite los operadores siguientes:

- **Select**: Las proyecciones se traducen a SELECT de SQL, incluida la construcción de objetos.
- **Where**: Los filtros se traducen en WHERE de SQL y compatible con la traducción entre `&&`, `||`, y `!` a los operadores SQL
- **SelectMany**: Permite desenredar las matrices a la cláusula SQL JOIN. Usar para encadenar o anidar expresiones para filtrar los elementos de la matriz.
- **OrderBy** y **OrderByDescending**: Traduce a ORDER BY con ASC o DESC.
- Los operadores **Count**, **Sum**, **Min**, **Max** y **Average** para la agregación y sus equivalentes asincrónicos **CountAsync**, **SumAsync**, **MinAsync**, **MaxAsync** y **AverageAsync**.
- **CompareTo**: Se traduce a las comparaciones de intervalos. Normalmente se utiliza para las cadenas, ya que éstos no son comparables en. NET.
- **Take**: Se traduce en la parte superior de SQL para limitar los resultados de una consulta.
- **Funciones matemáticas**: Admite la traducción desde .NET `Abs`, `Acos`, `Asin`, `Atan`, `Ceiling`, `Cos`, `Exp`, `Floor`, `Log`, `Log10`, `Pow`, `Round`, `Sign`, `Sin`, `Sqrt`, `Tan`, y `Truncate` a las funciones SQL integradas equivalentes.
- **Funciones de cadena**: Admite la traducción desde .NET `Concat`, `Contains`, `Count`, `EndsWith`,`IndexOf`, `Replace`, `Reverse`, `StartsWith`, `SubString`, `ToLower`, `ToUpper`, `TrimEnd`, y `TrimStart` a las funciones SQL integradas equivalentes.
- **Las funciones de matriz**: Admite la traducción desde .NET `Concat`, `Contains`, y `Count` a las funciones SQL integradas equivalentes.
- **Funciones de extensión geoespacial**: Admite la traducción desde los métodos auxiliares `Distance`, `IsValid`, `IsValidDetailed`, y `Within` a las funciones SQL integradas equivalentes.
- **Función de extensión de la función definida por el usuario**: Admite la traducción desde el método auxiliar `UserDefinedFunctionProvider.Invoke` a la función definida por el usuario correspondiente.
- **Varios**: Admite la traducción de `Coalesce` y operadores condicionales. Puede traducir `Contains` a String CONTAINS, ARRAY_CONTAINS o SQL IN, según el contexto.

### <a name="sql-query-operators"></a>Operadores de consulta SQL

Los ejemplos siguientes muestran cómo se traducen algunos de los operadores de consulta estándares de LINQ a consultas de Cosmos DB.

#### <a name="select-operator"></a>Seleccionar operador

La sintaxis es `input.Select(x => f(x))`, donde `f` es una expresión escalar.

**Seleccione el operador de ejemplo 1:**

- **Expresión lambda de LINQ**
  
  ```csharp
      input.Select(family => family.parents[0].familyName);
  ```
  
- **SQL** 
  
  ```sql
      SELECT VALUE f.parents[0].familyName
      FROM Families f
    ```
  
**Seleccione el operador de ejemplo 2:** 

- **Expresión lambda de LINQ**
  
  ```csharp
      input.Select(family => family.children[0].grade + c); // c is an int variable
  ```
  
- **SQL**
  
  ```sql
      SELECT VALUE f.children[0].grade + c
      FROM Families f
  ```
  
**Seleccione el operador de ejemplo 3:**

- **Expresión lambda de LINQ**
  
  ```csharp
    input.Select(family => new
    {
        name = family.children[0].familyName,
        grade = family.children[0].grade + 3
    });
  ```
  
- **SQL** 
  
  ```sql
      SELECT VALUE {"name":f.children[0].familyName,
                    "grade": f.children[0].grade + 3 }
      FROM Families f
  ```

#### <a name="selectmany-operator"></a>Operador SelectMany

La sintaxis es `input.SelectMany(x => f(x))`, donde `f` es una expresión escalar que devuelve un tipo de contenedor.

- **Expresión lambda de LINQ**
  
  ```csharp
      input.SelectMany(family => family.children);
  ```
  
- **SQL**

  ```sql
      SELECT VALUE child
      FROM child IN Families.children
  ```

#### <a name="where-operator"></a>Operador Where

La sintaxis es `input.Where(x => f(x))`, donde `f` es una expresión escalar que devuelve un valor booleano.

**En operador, el ejemplo 1:**

- **Expresión lambda de LINQ**
  
  ```csharp
      input.Where(family=> family.parents[0].familyName == "Wakefield");
  ```
  
- **SQL**
  
  ```sql
      SELECT *
      FROM Families f
      WHERE f.parents[0].familyName = "Wakefield"
  ```
  
**En operador, el ejemplo 2:**

- **Expresión lambda de LINQ**
  
  ```csharp
      input.Where(
          family => family.parents[0].familyName == "Wakefield" &&
          family.children[0].grade < 3);
  ```
  
- **SQL**
  
  ```sql
      SELECT *
      FROM Families f
      WHERE f.parents[0].familyName = "Wakefield"
      AND f.children[0].grade < 3
  ```

### <a name="composite-sql-queries"></a>Composición de consultas SQL

Puede componer los operadores anteriores para formar consultas más eficaces. Puesto que Cosmos DB admite contenedores anidados, puede concatenar o anidar la composición.

#### <a name="concatenation"></a>Concatenación

La sintaxis es `input(.|.SelectMany())(.Select()|.Where())*`. Una consulta concatenada puede empezar con un elemento opcional `SelectMany` consulta, seguida de varios `Select` o `Where` operadores.

**Concatenación, ejemplo 1:**

- **Expresión lambda de LINQ**
  
  ```csharp
      input.Select(family=>family.parents[0])
          .Where(familyName == "Wakefield");
  ```

- **SQL**
  
  ```sql
      SELECT *
      FROM Families f
      WHERE f.parents[0].familyName = "Wakefield"
  ```

**Ejemplo 2 de concatenación:**

- **Expresión lambda de LINQ**
  
  ```csharp
      input.Where(family => family.children[0].grade > 3)
          .Select(family => family.parents[0].familyName);
  ```

- **SQL**
  
  ```sql
      SELECT VALUE f.parents[0].familyName
      FROM Families f
      WHERE f.children[0].grade > 3
  ```

**Concatenación de ejemplo 3:**

- **Expresión lambda de LINQ**
  
  ```csharp
      input.Select(family => new { grade=family.children[0].grade}).
          Where(anon=> anon.grade < 3);
  ```
  
- **SQL**
  
  ```sql
      SELECT *
      FROM Families f
      WHERE ({grade: f.children[0].grade}.grade > 3)
  ```

**Concatenación de ejemplo 4:**

- **Expresión lambda de LINQ**
  
  ```csharp
      input.SelectMany(family => family.parents)
          .Where(parent => parents.familyName == "Wakefield");
  ```
  
- **SQL**
  
  ```sql
      SELECT *
      FROM p IN Families.parents
      WHERE p.familyName = "Wakefield"
  ```

#### <a name="nesting"></a>Anidamiento

La sintaxis es `input.SelectMany(x=>x.Q())` donde `Q` es un `Select`, `SelectMany`, o `Where` operador.

Una consulta anidada aplica la consulta interna a cada elemento del contenedor externo. Una característica importante es que la consulta interna puede hacer referencia a los campos de los elementos en el contenedor externo, como una autocombinación.

**Ejemplo 1 de anidamiento:**

- **Expresión lambda de LINQ**
  
  ```csharp
      input.SelectMany(family=>
          family.parents.Select(p => p.familyName));
  ```

- **SQL**
  
  ```sql
      SELECT VALUE p.familyName
      FROM Families f
      JOIN p IN f.parents
  ```

**Ejemplo 2 de anidamiento:**

- **Expresión lambda de LINQ**
  
  ```csharp
      input.SelectMany(family =>
          family.children.Where(child => child.familyName == "Jeff"));
  ```

- **SQL**
  
  ```sql
      SELECT *
      FROM Families f
      JOIN c IN f.children
      WHERE c.familyName = "Jeff"
  ```

**Ejemplo 3 de anidamiento:**

- **Expresión lambda de LINQ**
  
  ```csharp
      input.SelectMany(family => family.children.Where(
          child => child.familyName == family.parents[0].familyName));
  ```

- **SQL**
  
  ```sql
      SELECT *
      FROM Families f
      JOIN c IN f.children
      WHERE c.familyName = f.parents[0].familyName
  ```

## <a id="References"></a>Referencias

- [Especificación de SQL de Azure Cosmos DB](https://go.microsoft.com/fwlink/p/?LinkID=510612)
- [ANSI SQL 2011](https://www.iso.org/iso/iso_catalogue/catalogue_tc/catalogue_detail.htm?csnumber=53681)
- [JSON](https://json.org/)
- [Especificación de JavaScript](https://www.ecma-international.org/publications/standards/Ecma-262.htm) 
- [LINQ](/previous-versions/dotnet/articles/bb308959(v=msdn.10)) 
- Graefe, Goetz. [Técnicas de evaluación para grandes bases de datos de consulta](https://dl.acm.org/citation.cfm?id=152611). *Informática de encuestas ACM* 25, no. 2 (1993).
- Graefe, G. "El marco de cascadas para optimizar la consulta." *ENG de datos de IEEE. Alcista.* 18, no. 3 (1995).
- Lu, Ooi, Tan. "Procesamiento de consultas en sistemas de base de datos relacional paralela." *IEEE Computer Society presione* (1994).
- Olston, Christopher, Benjamin Reed, Utkarsh Srivastava, Ravi Kumar y Andrew Tomkins. "Pig Latin: Un lenguaje no forma externa para el procesamiento de datos." *SIGMOD* (2008).

## <a name="next-steps"></a>Pasos siguientes

- [Introducción a Azure Cosmos DB][introduction]
- [Ejemplos de .NET de Azure Cosmos DB](https://github.com/Azure/azure-cosmosdb-dotnet)
- [Niveles de coherencia de Azure Cosmos DB][consistency-levels]

[1]: ./media/how-to-sql-query/sql-query1.png
[introduction]: introduction.md
[consistency-levels]: consistency-levels.md
