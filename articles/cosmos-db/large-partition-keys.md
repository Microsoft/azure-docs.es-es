---
title: Crear contenedores de Azure Cosmos con clave de partición de gran tamaño mediante el portal de Azure y SDK.
description: Obtenga información sobre cómo crear un contenedor en Azure Cosmos DB con la clave de partición grande mediante Azure portal y diferentes SDK.
author: markjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 05/06/2019
ms.author: mjbrown
ms.openlocfilehash: 322d5630daeedaa671f994b1374f15f655811de5
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65796083"
---
# <a name="create-containers-with-large-partition-key"></a>Crear contenedores con la clave de partición de gran tamaño

Azure Cosmos DB usa el esquema de partición basado en hash para conseguir un escalado horizontal de datos. Todos los contenedores de Azure Cosmos creados antes del 3 de mayo de 2019 utilizan una función hash que se calcula en función de los primeros 100 bytes de la clave de partición de hash. Si hay varias claves de partición que tengan el mismo los primeros 100 bytes, a continuación, esas particiones lógicas se consideran la misma partición lógica por el servicio. Esto puede causar problemas, como la cuota de tamaño de partición que se va a incorrecto y los índices únicos que se va a aplicar a través de las claves de partición. Para solucionar este problema, se introducen claves de partición grande. Azure Cosmos DB ahora admite claves de partición de gran tamaño con valores de hasta 2 KB. 

Hasta 2 KB de claves de partición grande se admiten las claves mediante la funcionalidad de una versión mejorada de la función hash, que puede generar un hash único de la partición de gran tamaño. Esta versión de hash también se recomienda para escenarios con la cardinalidad de clave de partición alta independientemente del tamaño de la clave de partición. Una cardinalidad de clave de partición se define como el número de particiones únicas de lógicas, por ejemplo, en el orden de ~ 30000 particiones lógicas en un contenedor. En este artículo se describe cómo crear un contenedor con una clave de partición de gran tamaño mediante el portal de Azure y los diferentes SDK. 

## <a name="create-a-large-partition-key-net-sdk-v2"></a>Crear una clave de partición de gran tamaño (.Net SDK V2)

Cuando se usa el SDK de .net para crear un contenedor con la clave de partición de gran tamaño, debe especificar el `PartitionKeyDefinitionVersion.V2` propiedad. El ejemplo siguiente muestra cómo especificar la propiedad de versión dentro del objeto PartitionKeyDefinition y establézcalo como PartitionKeyDefinitionVersion.V2:

```csharp
DocumentCollection collection = await newClient.CreateDocumentCollectionAsync(
database,
     new DocumentCollection
        {
           Id = Guid.NewGuid().ToString(),
           PartitionKey = new PartitionKeyDefinition
           {
             Paths = new Collection<string> {"/longpartitionkey" },
             Version = PartitionKeyDefinitionVersion.V2
           }
         },
      new RequestOptions { OfferThroughput = 400 });
```

## <a name="create-a-large-partition-key-azure-portal"></a>Crear una clave de partición de gran tamaño (Azure portal) 

Para crear una clave de partición de gran tamaño, mientras crea un nuevo contenedor mediante el portal de Azure, compruebe el **mi clave de partición es mayor que 100 bytes** opción. De forma predeterminada, los nuevos contenedores participarán en el uso de las claves de partición de gran tamaño. Desactive la casilla si no necesita las claves de partición de gran tamaño o si tiene aplicaciones que se ejecutan en la versión de SDK anterior a 1,18.

![Crear claves de partición grande mediante Azure portal](./media/large-partition-keys/large-partition-key-with-portal.png)


## <a name="supported-sdk-versions"></a>Versiones de SDK compatibles

Las claves de partición de gran tamaño son compatibles con las siguientes versiones mínimas de SDK:

|Tipo SDK  | Versión mínima   |
|---------|---------|
|.Net     |    1.18     |
|Sincronización de Java     |   2.4.0      |
|Java Async   |  2.5.0        |
| API DE REST | versión más alta que `2017-05-03` utilizando el `x-ms-version` encabezado de solicitud.|
 
## <a name="next-steps"></a>Pasos siguientes

* [Creación de particiones en Azure Cosmos DB](partitioning-overview.md)
* [Unidades de solicitud en Azure Cosmos DB](request-units.md)
* [Aprovisionar rendimiento en contenedores y bases de datos](set-throughput.md)
* [Uso de la cuenta de Azure Cosmos](account-overview.md)


