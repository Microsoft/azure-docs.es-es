---
title: Introducción a las consultas SQL en Azure Cosmos DB
description: Obtenga información sobre cómo usar consultas SQL para consultar datos de Azure Cosmos DB. Puede cargar datos de ejemplo en un contenedor en Azure Cosmos DB y consultarlos.
author: timsander1
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 07/24/2020
ms.author: tisande
ms.openlocfilehash: d292b7cfcda73cb4cd6ac2535c7e27fc675e1030
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87308192"
---
# <a name="getting-started-with-sql-queries"></a>Introducción a las consultas SQL

En las cuentas de la API de SQL de Azure Cosmos DB, hay dos maneras de leer los datos:

**Lecturas puntuales**: puede realizar una búsqueda de clave y valor en un único *id. de elemento* y clave de partición. La combinación de *id. de elemento* y clave de partición es la clave y el propio elemento es el valor. En el caso de un documento de 1 KB, las lecturas puntuales normalmente cuestan 1 [unidad de solicitud](request-units.md) con una latencia inferior a 10 ms. Las lecturas puntuales devuelven un único elemento.

**Consultas SQL**: puede consultar datos escribiendo consultas mediante el Lenguaje de consulta estructurado (SQL) como lenguaje de consulta JSON. Las consultas siempre cuestan al menos 2,3 unidades de solicitud y, en general, tendrán una latencia mayor y más variable que las lecturas puntuales. Las consultas pueden devolver muchos elementos.

La mayoría de las cargas de trabajo con mucha actividad de lectura en Azure Cosmos DB utilizan una combinación de lecturas puntuales y consultas SQL. Si solo necesita leer un único elemento, las lecturas puntuales son más baratas y rápidas que las consultas. Las lecturas puntuales no necesitan usar el motor de consultas para tener acceso a los datos y pueden leer los datos directamente. Por supuesto, no es posible que todas las cargas de trabajo lean los datos exclusivamente mediante lecturas puntuales, por lo que la compatibilidad con SQL como lenguaje de consulta y la [indexación independiente del esquema](index-overview.md) proporcionan una manera más flexible de acceder a los datos.

Estos son algunos ejemplos de cómo realizar lecturas puntuales con cada SDK:

- [SDK de .NET](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync?view=azure-dotnet)
- [SDK de Java](https://docs.microsoft.com/java/api/com.azure.cosmos.cosmoscontainer.readitem?view=azure-java-stable#com_azure_cosmos_CosmosContainer__T_readItem_java_lang_String_com_azure_cosmos_models_PartitionKey_com_azure_cosmos_models_CosmosItemRequestOptions_java_lang_Class_T__)
- [SDK de Node.js](https://docs.microsoft.com/javascript/api/@azure/cosmos/item?view=azure-node-latest#read-requestoptions-)
- [SDK de Python](https://docs.microsoft.com/python/api/azure-cosmos/azure.cosmos.containerproxy?view=azure-python#read-item-item--partition-key--populate-query-metrics-none--post-trigger-include-none----kwargs-)

En el resto de este documento se muestra cómo empezar a escribir consultas SQL en Azure Cosmos DB. Las consultas SQL se pueden ejecutar mediante el SDK o Azure Portal.

## <a name="upload-sample-data"></a>Cargar datos de ejemplo

En la cuenta de la API de SQL para Cosmos DB, cree un contenedor denominado `Families`. Cree dos elementos JSON sencillos en el contenedor. Puede ejecutar la mayoría de las consultas de ejemplo en la documentación de consultas de Azure Cosmos DB con este conjunto de datos.

### <a name="create-json-items"></a>Creación de elementos JSON

Con el siguiente código se crean dos elementos JSON sencillos sobre familias. Los elementos JSON sencillos para las familias Andersen y Wakefield incluyen a los padres, los hijos, sus mascotas, la dirección y la información de registro. El primer elemento tiene cadenas, números, valores booleanos, matrices y propiedades anidadas.

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

El segundo elemento usa `givenName` y `familyName` en lugar de `firstName` y `lastName`.

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

### <a name="query-the-json-items"></a>Consulta de los elementos JSON

Pruebe algunas consultas con los datos JSON para entender algunos aspectos clave del lenguaje de consulta SQL de Azure Cosmos DB.

La consulta siguiente devuelve los elementos en los que el campo `id` coincide con `AndersenFamily`. Puesto que es una consulta `SELECT *`, la salida de la consulta es el elemento JSON completo. Para más información sobre la sintaxis de SELECT, consulte [Instrucción SELECT](sql-query-select.md).

```sql
    SELECT *
    FROM Families f
    WHERE f.id = "AndersenFamily"
```

Los resultados de la consulta son:

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

La siguiente consulta vuelve a formatear la salida JSON en una forma diferente. La consulta proyecta un nuevo objeto JSON `Family` con dos campos seleccionados, `Name` y `City`, cuando la ciudad de la dirección es la misma que el estado. "NY, NY" coincide con esto.

```sql
    SELECT {"Name":f.id, "City":f.address.city} AS Family
    FROM Families f
    WHERE f.address.city = f.address.state
```

Los resultados de la consulta son:

```json
    [{
        "Family": {
            "Name": "WakefieldFamily",
            "City": "NY"
        }
    }]
```

La consulta siguiente devuelve todos los nombres proporcionados de los hijos de la familia cuyo `id` coincide con `WakefieldFamily`, ordenados por ciudad.

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

## <a name="remarks"></a>Observaciones

Los ejemplos anteriores muestran varios aspectos del lenguaje de consulta de Cosmos DB:  

* Como la API de SQL trabaja en valores JSON, trata con entidades en forma de árbol en lugar de con filas y columnas. Puede hacer referencia a los nodos de árbol de cualquier profundidad arbitraria, como `Node1.Node2.Node3…..Nodem`, de forma similar a la referencia de dos partes de `<table>.<column>` en SQL ANSI.

* Dado que el lenguaje de consulta funciona con datos sin esquema, el sistema de tipos debe enlazarse dinámicamente. La misma expresión podría generar diversos tipos en distintos elementos. El resultado de una consulta es un valor JSON válido, pero no se garantiza que sea de un esquema fijo.  

* Azure Cosmos DB solo admite elementos JSON estrictos. El sistema de tipos y las expresiones se restringen para tratar únicamente con tipos JSON. Para más información, consulte la [especificación de JSON](https://www.json.org/).  

* Un contenedor de Cosmos es una colección sin esquemas de elementos JSON. Las relaciones tanto dentro de los elementos de un contenedor como entre ellos se capturan de manera implícita por independencia, no por las relaciones entre la clave principal y la clave externa. Esta característica es importante para las combinaciones dentro del elemento descritas más adelante en este artículo.

## <a name="next-steps"></a>Pasos siguientes

- [Introducción a Azure Cosmos DB](introduction.md)
- [Ejemplos de .NET de Azure Cosmos DB](https://github.com/Azure/azure-cosmos-dotnet-v3)
- [Cláusula SELECT](sql-query-select.md)
