---
title: Solución de problemas de excepciones de recurso de Azure Cosmos DB no encontrado
description: Descubra cómo diagnosticar y corregir excepciones de recurso no encontrado.
author: j82w
ms.service: cosmos-db
ms.date: 07/13/2020
ms.author: jawilley
ms.topic: troubleshooting
ms.reviewer: sngun
ms.openlocfilehash: 1d778b4330389d23b0fe7179a005abfbd7d66d5c
ms.sourcegitcommit: 927dd0e3d44d48b413b446384214f4661f33db04
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88871112"
---
# <a name="diagnose-and-troubleshoot-azure-cosmos-db-not-found-exceptions"></a>Diagnóstico y solución de problemas de excepciones recurso de Azure Cosmos DB no encontrado
El código de estado HTTP 404 representa que el recurso ya no existe.

## <a name="expected-behavior"></a>Comportamiento esperado
Hay muchos escenarios válidos en los que una aplicación espera un código 404 y administra correctamente el escenario.

## <a name="a-not-found-exception-was-returned-for-an-item-that-should-exist-or-does-exist"></a>Se ha devuelto la excepción de recurso no encontrado para un elemento que existe o debería existir
A continuación se ofrecen las posibles razones para que se devuelva un código de estado 404 cuando el elemento existe o debería existir.

### <a name="race-condition"></a>Condición de carrera
Hay varias instancias de cliente de SDK y la lectura se produjo antes de la escritura.

#### <a name="solution"></a>Solución:
1. La coherencia de la cuenta predeterminada para Azure Cosmos DB es la coherencia de la sesión. Cuando se crea o se actualiza un elemento, la respuesta devolverá un token de sesión que se puede pasar de una instancia de SDK a otra para garantizar que la solicitud de lectura lea desde una réplica que tiene ese cambio.
1. Cambie el [nivel de coherencia](consistency-levels-choosing.md) a un [nivel más seguro](consistency-levels-tradeoffs.md).

### <a name="invalid-partition-key-and-id-combination"></a>Combinación no válida de clave de partición e identificador
La combinación de la clave de partición y el identificador no es válida.

#### <a name="solution"></a>Solución:
Corrija la lógica de aplicación que provoca la combinación incorrecta. 

### <a name="invalid-character-in-an-item-id"></a>Carácter no válido en un identificador de elemento
Se ha insertado un elemento en Cosmos DB con un [carácter no válido](https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id?view=azure-dotnet#remarks) en el identificador de elemento.

#### <a name="solution"></a>Solución:
Cambie el identificador a un valor que no contenga el carácter especial. Si cambiar de identificador no es una alternativa, puede codificar el identificador en Base64 para omitir los caracteres especiales.

Los elementos ya insertados en el contenedor para el identificador se pueden reemplazar con valores RID en lugar de referencias basadas en nombres.
```c#
// Get a container reference that uses RID values.
ContainerProperties containerProperties = await this.Container.ReadContainerAsync();
string[] selfLinkSegments = containerProperties.SelfLink.Split('/');
string databaseRid = selfLinkSegments[1];
string containerRid = selfLinkSegments[3];
Container containerByRid = this.cosmosClient.GetContainer(databaseRid, containerRid);

// Invalid characters are listed here.
//https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id?view=azure-dotnet#remarks
FeedIterator<JObject> invalidItemsIterator = this.Container.GetItemQueryIterator<JObject>(
    @"select * from t where CONTAINS(t.id, ""/"") or CONTAINS(t.id, ""#"") or CONTAINS(t.id, ""?"") or CONTAINS(t.id, ""\\"") ");
while (invalidItemsIterator.HasMoreResults)
{
    foreach (JObject itemWithInvalidId in await invalidItemsIterator.ReadNextAsync())
    {
        // Choose a new ID that doesn't contain special characters.
        // If that isn't possible, then Base64 encode the ID to escape the special characters.
        byte[] plainTextBytes = Encoding.UTF8.GetBytes(itemWithInvalidId["id"].ToString());
        itemWithInvalidId["id"] = Convert.ToBase64String(plainTextBytes);

        // Update the item with the new ID value by using the RID-based container reference.
        JObject item = await containerByRid.ReplaceItemAsync<JObject>(
            item: itemWithInvalidId,
            ID: itemWithInvalidId["_rid"].ToString(),
            partitionKey: new Cosmos.PartitionKey(itemWithInvalidId["status"].ToString()));

        // Validating the new ID can be read by using the original name-based container reference.
        await this.Container.ReadItemAsync<ToDoActivity>(
            item["id"].ToString(),
            new Cosmos.PartitionKey(item["status"].ToString())); ;
    }
}
```

### <a name="time-to-live-purge"></a>Purga de la propiedad Período de vida
El elemento tenía definida la propiedad [Período de vida (TTL)](https://docs.microsoft.com/azure/cosmos-db/time-to-live). El elemento se purgó porque la propiedad TTL expiró.

#### <a name="solution"></a>Solución:
Cambie la propiedad TTL para evitar que el elemento se purgue.

### <a name="lazy-indexing"></a>Indexación diferida
La [indexación diferida](index-policy.md#indexing-mode) no se ha actualizado.

#### <a name="solution"></a>Solución:
Espere a que la indexación se actualice o cambie la directiva de indexación.

### <a name="parent-resource-deleted"></a>Recurso primario eliminado
Se eliminó la base de datos o el contenedor donde se encuentra el elemento.

#### <a name="solution"></a>Solución:
1. [Restaure](https://docs.microsoft.com/azure/cosmos-db/online-backup-and-restore#backup-retention-period) el recurso primario o vuelva a crearlo.
1. Cree un nuevo recurso para reemplazar el recurso eliminado.

## <a name="next-steps"></a>Pasos siguientes
* [Diagnóstico y solución de problemas](troubleshoot-dot-net-sdk.md) al utilizar el SDK de Azure Cosmos DB para .NET.
* Más información sobre las directrices de rendimiento de [.NET v3](performance-tips-dotnet-sdk-v3-sql.md) y [.NET v2](performance-tips.md).