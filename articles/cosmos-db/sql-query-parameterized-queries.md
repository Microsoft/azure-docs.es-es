---
title: Consultas parametrizadas en Azure Cosmos DB
description: Aprenda el modo en que las consultas de SQL con parámetros permiten controlar y evitar de forma coherente la entrada por parte de los usuarios, e impiden la exposición accidental de datos mediante la inyección de código SQL.
author: timsander1
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 07/29/2020
ms.author: tisande
ms.openlocfilehash: f66bc89ef56dd0c2291903d531a4637210abd8df
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87496991"
---
# <a name="parameterized-queries-in-azure-cosmos-db"></a>Consultas parametrizadas en Azure Cosmos DB

Azure Cosmos DB admite consultas con parámetros que se expresen con la notación @ ya conocida. El uso de SQL con parámetros permite controlar y evitar de forma coherente la entrada por parte de los usuarios, e impide la exposición accidental de datos a través de la inyección de código SQL.

## <a name="examples"></a>Ejemplos

Por ejemplo, puede escribir una consulta que tome `lastName` y `address.state` como parámetros, y ejecutarla para distintos valores de `lastName` y `address.state` basándose en la entrada del usuario.

```sql
    SELECT *
    FROM Families f
    WHERE f.lastName = @lastName AND f.address.state = @addressState
```

Luego puede enviar esta solicitud a Azure Cosmos DB como consulta JSON parametrizada, como la siguiente:

```sql
    {
        "query": "SELECT * FROM Families f WHERE f.lastName = @lastName AND f.address.state = @addressState",
        "parameters": [
            {"name": "@lastName", "value": "Wakefield"},
            {"name": "@addressState", "value": "NY"},
        ]
    }
```

En el ejemplo siguiente se establece el argumento TOP con una consulta parametrizada:

```sql
    {
        "query": "SELECT TOP @n * FROM Families",
        "parameters": [
            {"name": "@n", "value": 10},
        ]
    }
```

Los valores de los parámetros pueden ser cualquier tipo de JSON válido: cadenas, números, booleanos, null o incluso matrices o JSON anidado. Dado que Azure Cosmos DB no tiene ningún esquema, los parámetros no se validan en función de ningún tipo.

Estos son algunos ejemplos de consultas con parámetros en cada SDK de Azure Cosmos DB:

- [SDK de .NET](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/Queries/Program.cs#L195)
- [Java](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L392-L421)
- [Node.js](https://github.com/Azure/azure-cosmos-js/blob/master/samples/ItemManagement.ts#L58-L79)
- [Python](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/document_management.py#L66-L78)

## <a name="next-steps"></a>Pasos siguientes

- [Ejemplos de .NET de Azure Cosmos DB](https://github.com/Azure/azure-cosmos-dotnet-v3)
- [Modelado de datos de documentos](modeling-data.md)
