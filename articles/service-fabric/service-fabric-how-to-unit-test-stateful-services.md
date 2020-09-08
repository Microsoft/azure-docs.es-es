---
title: Desarrollo de pruebas unitarias para servicios con estado
description: Más información sobre las pruebas unitarias en Azure Service Fabric para servicios con estado y consideraciones especiales que debe tener en cuenta durante el desarrollo.
ms.topic: conceptual
ms.date: 09/04/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: 287c0544daa3c44d91fd336b502c496b9b4bb266
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89011402"
---
# <a name="create-unit-tests-for-stateful-services"></a>Creación de pruebas unitarias para servicios con estado
Las pruebas unitarias para los servicios con estado de Service Fabric detectan errores comunes que no necesariamente detectaría una prueba unitaria convencional específica para dominios o aplicaciones. Durante el desarrollo de pruebas unitarias para los servicios con estado, hay algunas consideraciones especiales que deben tenerse en cuenta.

1. Cada réplica ejecuta código de la aplicación, pero en un contexto diferente. Si el servicio utiliza tres réplicas, el código de servicio se ejecuta de forma paralela en tres nodos con contextos y roles distintos.
2. El estado almacenado dentro del servicio con estado debe ser coherente entre todas las réplicas. El administrador de estado y las colecciones de confianza proporcionarán esta coherencia de forma nativa. Sin embargo, el estado en memoria se debe administrar mediante el código de aplicación.
3. Cada réplica cambiará de rol en algún momento durante su ejecución en el clúster. Si el nodo que hospeda a una réplica principal deja de estar disponible o se sobrecarga, la réplica secundaria se volverá principal. Este es el comportamiento natural para Service Fabric; por lo tanto, los servicios deben planificar que en algún momento se ejecutarán con un rol diferente.

En este artículo se da por sentado que ya ha leído [Unit testing stateful services in Service Fabric](service-fabric-concepts-unit-testing.md) (Pruebas unitarias de servicios con estado en Service Fabric).

## <a name="the-servicefabricmocks-library"></a>La biblioteca ServiceFabric.Mocks
A partir de la versión 3.3.0, [ServiceFabric.Mocks](https://www.nuget.org/packages/ServiceFabric.Mocks/) proporciona una API para simular la orquestación de réplicas y la administración de estados. Esta es la que usará en los ejemplos.

[Nuget](https://www.nuget.org/packages/ServiceFabric.Mocks/)
[GitHub](https://github.com/loekd/ServiceFabric.Mocks)

*Microsoft no es propietario de ServiceFabric.Mocks ni le da mantenimiento. Sin embargo, actualmente es la biblioteca recomendada por Microsoft para las pruebas unitarias de servicios con estado.*

## <a name="set-up-the-mock-orchestration-and-state"></a>Configuración de la simulación de orquestación y estado
Como parte de la organización de una prueba, se creará un conjunto de réplicas ficticias y un administrador de estados. El conjunto de réplicas podrá crear una instancia del servicio probado para cada réplica. También podrá ejecutar eventos de ciclo de vida como `OnChangeRole` y `RunAsync`. El administrador de estados ficticios garantizará que cualquier operación realizada en comparación con el administrador de estados se ejecute y se mantenga como lo haría el administrador de estados real.

1. Cree a un delegado de generador de servicios que creará una instancia del servicio que se está probando. Debe ser similar o idéntico a la devolución de llamada del generador de servicios que se encuentra normalmente en `Program.cs` para un actor o un servicio de Service Fabric. Esto debe ir después de la firma siguiente:
   ```csharp
   MyStatefulService CreateMyStatefulService(StatefulServiceContext context, IReliableStateManagerReplica2 stateManager)
   ```
2. Cree una instancia de la clase `MockReliableStateManager`. Así se simularán todas las interacciones con el administrador de estados.
3. Cree una instancia de `MockStatefulServiceReplicaSet<TStatefulService>` en la que `TStatefulService` sea el tipo de servicio que se está probando. Para ello, necesitará el delegado que creó en el paso 1 y el administrador de estados del que creó una instancia en el paso 2
4. Agregue réplicas al conjunto de réplicas. Especifique el rol (por ejemplo, Primary, ActiveSecondary, IdleSecondary) y el identificador de réplica
   > Guarde los id. de réplica. Es probable que se usen durante las partes de ejecución y aserción.

```csharp
//service factory to instruct how to create the service instance
var serviceFactory = (StatefulServiceContext context, IReliableStateManagerReplica2 stateManager) => new MyStatefulService(context, stateManager);
//instantiate a new mock state manager
var stateManager = new MockReliableStateManager();
//instantiate a new replica set with the service factory and state manager
var replicaSet = new MockStatefulServiceReplicaSet<MyStatefulService>(CreateStatefulService, stateManager);
//add a new Primary replica with id 1
await replicaSet.AddReplicaAsync(ReplicaRole.Primary, 1);
//add a new ActiveSecondary replica with id 2
await replicaSet.AddReplicaAsync(ReplicaRole.ActiveSecondary, 2);
//add a second ActiveSecondary replica with id 3
await replicaSet.AddReplicaAsync(ReplicaRole.ActiveSecondary, 3);
```

## <a name="execute-service-requests"></a>Ejecución de las solicitudes de servicio
Las solicitudes de servicio se pueden ejecutar en una réplica específica mediante las propiedades y búsquedas de conveniencia.
```csharp
const string stateName = "test";
var payload = new Payload(StatePayload);

//execute a request on the primary replica using
await replicaSet.Primary.ServiceInstance.InsertAsync(stateName, payload);

//execute a request against replica with id 2
await replicaSet[2].ServiceInstance.InsertAsync(stateName, payload);

//execute a request against one of the active secondary replicas
await replicaSet.FirstActiveSecondary.InsertAsync(stateName, payload);
```

## <a name="execute-a-service-move"></a>Ejecución de una migración de servicio
El conjunto de réplicas ficticio expone varios métodos útiles para desencadenar diferentes tipos de migraciones de servicio.
```csharp
//promote the first active secondary to primary
replicaSet.PromoteNewReplicaToPrimaryAsync();
//promote the secondary with replica id 4 to primary
replicaSet.PromoteNewReplicaToPrimaryAsync(4);

//promote the first idle secondary to an active secondary
PromoteIdleSecondaryToActiveSecondaryAsync();
//promote idle secondary with replica id 4 to active secondary
PromoteIdleSecondaryToActiveSecondaryAsync(4);

//add a new replica with randomly assigned replica id and promote it to primary
PromoteNewReplicaToPrimaryAsync()
//add a new replica with replica id 4 and promote it to primary
PromoteNewReplicaToPrimaryAsync(4)
```

## <a name="putting-it-all-together"></a>Resumen
La siguiente prueba muestra cómo configurar un conjunto de réplicas de tres nodos y comprobar que los datos están disponibles desde una réplica secundaria después de un cambio de rol. Posiblemente se detecte la incidencia típica en la que los datos agregados durante `InsertAsync` se guardaron en algo en la memoria o en una colección de confianza sin ejecutar `CommitAsync`. En ambos casos, la réplica secundaria no estaría sincronizada con la principal. Esto daría lugar a respuestas incoherentes después de migrar el servicio.

```csharp
[TestMethod]
public async Task TestServiceState_InMemoryState_PromoteActiveSecondary()
{
    var stateManager = new MockReliableStateManager();
    var replicaSet = new MockStatefulServiceReplicaSet<MyStatefulService>(CreateStatefulService, stateManager);
    await replicaSet.AddReplicaAsync(ReplicaRole.Primary, 1);
    await replicaSet.AddReplicaAsync(ReplicaRole.ActiveSecondary, 2);
    await replicaSet.AddReplicaAsync(ReplicaRole.ActiveSecondary, 3);

    const string stateName = "test";
    var payload = new Payload(StatePayload);

    //insert data
    await replicaSet.Primary.ServiceInstance.InsertAsync(stateName, payload);
    //promote one of the secondaries to primary
    await replicaSet.PromoteActiveSecondaryToPrimaryAsync(2);
    //get data
    var payloads = (await replicaSet.Primary.ServiceInstance.GetPayloadsAsync()).ToList();

    //data should match what was inserted against the primary
    Assert.IsTrue(payloads.Count == 1);
    Assert.IsTrue(payloads[0].Content == payload.Content);

    //verify the data was saved against the reliable dictionary
    var dictionary = await StateManager.GetOrAddAsync<IReliableDictionary<string, Payload>>(MyStatefulService.StateManagerDictionaryKey);
    using(var tx = StateManager.CreateTransaction())
    {
        var payload = await dictionary.TryGetValue(stateName);
        Assert.IsTrue(payload.HasValue);
        Assert.IsTrue(payload.Value.Content == payload.Content);
    }
}
```

## <a name="next-steps"></a>Pasos siguientes
Obtenga información sobre cómo probar la [comunicación servicio a servicio](service-fabric-testability-scenarios-service-communication.md) y [simular errores mediante caos controlado](service-fabric-controlled-chaos.md).
