---
title: Solución de problemas de la excepción de recurso no encontrado de Azure Cosmos DB
description: Diagnóstico y corrección de la excepción de recurso no encontrado
author: j82w
ms.service: cosmos-db
ms.date: 07/13/2020
ms.author: jawilley
ms.topic: troubleshooting
ms.reviewer: sngun
ms.openlocfilehash: f391772672904a1e2bdfa0a741d9f85a6edeea8b
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87293901"
---
# <a name="diagnose-and-troubleshoot-azure-cosmos-db-not-found"></a>Diagnóstico y solución de problemas de recurso no encontrado de Azure Cosmos DB
El código de estado HTTP 404 representa que el recurso ya no existe.

## <a name="expected-behavior"></a>Comportamiento esperado
Hay muchos escenarios válidos en los que una aplicación espera un error 404 y controla correctamente el escenario.

## <a name="not-found-was-returned-for-an-item-that-should-exist-or-does-exist"></a>La excepción No encontrado se devuelve para un elemento que existe o debería existir
A continuación se muestra la posible razón para que se devuelva un código de estado 404 si el elemento existe o debería existir.

### <a name="1-race-condition"></a>1. Condición de carrera
Hay varias instancias de cliente de SDK y la lectura se produjo antes de la escritura.

#### <a name="solution"></a>Solución:
1. La coherencia de la cuenta predeterminada para Cosmos DB es la coherencia de la sesión. Cuando se crea o se actualiza un elemento, la respuesta devolverá un token de sesión que se puede pasar entre instancias del SDK para garantizar que la solicitud de lectura lee de una réplica con ese cambio.
2. Cambie el [nivel de coherencia](consistency-levels-choosing.md) a un [nivel más seguro](consistency-levels-tradeoffs.md).

### <a name="2-invalid-partition-key-and-id-combination"></a>2. Combinación de clave de partición e identificador no válida
La combinación de clave de partición e identificador no es válida.

#### <a name="solution"></a>Solución:
Corrija la lógica de aplicación que está causando la combinación incorrecta. 

### <a name="3-invalid-character-in-item-id"></a>3. Carácter no válido en el identificador del elemento
Un elemento se ha insertado en Cosmos DB con un [carácter no válido](https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id?view=azure-dotnet#remarks) en el identificador del elemento.

#### <a name="solution"></a>Solución:
Se recomienda que los usuarios cambien el identificador a un valor diferente que no contenga los caracteres especiales. Si el cambio de identificador no es una opción viable, puede codificar en Base64 el identificador para escapar los caracteres especiales.

Los elementos ya insertados en el contenedor se pueden reemplazar con valores RID en lugar de referencias basadas en nombres.
```c#
// Get a container reference that use RID values
ContainerProperties containerProperties = await this.Container.ReadContainerAsync();
string[] selfLinkSegments = containerProperties.SelfLink.Split('/');
string databaseRid = selfLinkSegments[1];
string containerRid = selfLinkSegments[3];
Container containerByRid = this.cosmosClient.GetContainer(databaseRid, containerRid);

// List of invalid characters are listed here.
//https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id?view=azure-dotnet#remarks
FeedIterator<JObject> invalidItemsIterator = this.Container.GetItemQueryIterator<JObject>(
    @"select * from t where CONTAINS(t.id, ""/"") or CONTAINS(t.id, ""#"") or CONTAINS(t.id, ""?"") or CONTAINS(t.id, ""\\"") ");
while (invalidItemsIterator.HasMoreResults)
{
    foreach (JObject itemWithInvalidId in await invalidItemsIterator.ReadNextAsync())
    {
        // It recommend to chose a new ID that does not contain special characters, but
        // if that is not possible then it can be Base64 encoded to escape the special characters
        byte[] plainTextBytes = Encoding.UTF8.GetBytes(itemWithInvalidId["id"].ToString());
        itemWithInvalidId["id"] = Convert.ToBase64String(plainTextBytes);

        // Update the item with the new ID value using the rid based container reference
        JObject item = await containerByRid.ReplaceItemAsync<JObject>(
            item: itemWithInvalidId,
            ID: itemWithInvalidId["_rid"].ToString(),
            partitionKey: new Cosmos.PartitionKey(itemWithInvalidId["status"].ToString()));

        // Validate the new ID can be read using the original name based contianer reference
        await this.Container.ReadItemAsync<ToDoActivity>(
            item["id"].ToString(),
            new Cosmos.PartitionKey(item["status"].ToString())); ;
    }
}
```

### <a name="4-time-to-live-ttl-purge"></a>4. Purga del período de vida (TTL)
El elemento tenía definida la propiedad [Período de vida (TTL)](https://docs.microsoft.com/azure/cosmos-db/time-to-live). El elemento se purgó porque expiró el período de vida.

#### <a name="solution"></a>Solución:
Cambie el período de vida para evitar que el elemento se purgue.

### <a name="5-lazy-indexing"></a>5. Indexación diferida
La [indexación diferida](index-policy.md#indexing-mode) no se ha puesto al día.

#### <a name="solution"></a>Solución:
Espere a que la indexación se ponga al día o cambie la directiva de indexación

### <a name="6-parent-resource-deleted"></a>6. Recurso primario eliminado
La base de datos o el contenedor en el que se encuentra el elemento se ha eliminado.

#### <a name="solution"></a>Solución:
1. [Restaure](https://docs.microsoft.com/azure/cosmos-db/online-backup-and-restore#backup-retention-period) el recurso primario o vuelva a crear los recursos.
2. Cree un nuevo recurso para reemplazar el recurso eliminado.

## <a name="next-steps"></a>Pasos siguientes
* [Diagnóstico y solución de problemas](troubleshoot-dot-net-sdk.md) al usar el SDK de .NET de Azure Cosmos DB
* Más información sobre las directrices de rendimiento para [.NET V3](performance-tips-dotnet-sdk-v3-sql.md) y [.NET V2](performance-tips.md)