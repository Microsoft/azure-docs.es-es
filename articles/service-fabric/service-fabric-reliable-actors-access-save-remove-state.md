---
title: Administrar el estado de Azure Service Fabric
description: Obtenga información sobre el acceso, el guardado y la eliminación del estado de un Reliable Actor de Azure Service Fabric y sobre las consideraciones que se deben tener en cuenta al diseñar una aplicación.
author: vturecek
ms.topic: conceptual
ms.date: 03/19/2018
ms.author: vturecek
ms.custom: devx-track-csharp
ms.openlocfilehash: e97fadb374fc45110f5071aab48da0a57eef4284
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89012796"
---
# <a name="access-save-and-remove-reliable-actors-state"></a>Acceder al estado de Reliable Actors, guardarlo y quitarlo
[Reliable Actors](service-fabric-reliable-actors-introduction.md) son objetos de subproceso único que encapsulan la lógica y el estado, y mantienen el estado de forma confiable. Cada instancia de actor tiene su propio [administrador de estado](service-fabric-reliable-actors-state-management.md): una estructura de datos similar a un diccionario que almacena de manera confiable los pares clave-valor. El administrador de estado es un contenedor alrededor de un proveedor de estado. Se puede utilizar para almacenar datos, con independencia de la [configuración de persistencia](service-fabric-reliable-actors-state-management.md#state-persistence-and-replication) que se utilice.

Las claves del administrador de estado deben ser cadenas. Los valores son genéricos y pueden ser de cualquier tipo, incluidos los tipos personalizados. Los valores almacenados en el administrador de estado deben ser serializables en el contrato de datos, ya que pueden transmitirse a través de la red a otros nodos durante la replicación y pueden escribirse en el disco, dependiendo de la configuración de persistencia del estado de un actor.

El administrador de estado expone los métodos de diccionario comunes para administrar el estado, de manera similar a los que se encuentran en un diccionario confiable.

Para obtener información, consulte [los procedimientos recomendados para administrar el estado de actor](service-fabric-reliable-actors-state-management.md#best-practices).

## <a name="access-state"></a>Estado de acceso
Se accede al estado a través del administrador de estado por clave. Los métodos del administrador de estado son completamente asincrónicos, ya que pueden requerir una E/S de disco cuando los actores tienen un estado persistente. Después del primer acceso, los objetos de estado se almacenan en caché en la memoria. Las operaciones de acceso repetidas acceden a los objetos directamente desde la memoria y devuelven sincrónicamente sin incurrir en la sobrecarga de intercambio de contexto asincrónico o de E/S de disco. Se quita un objeto de estado de la memoria caché en los casos siguientes:

* Un método de actor produce una excepción no controlada después de recuperar un objeto del administrador de estado.
* Un actor se vuelve a activar después de haberse desactivado o después de un error.
* El proveedor de estado pagina el estado en el disco. Este comportamiento depende de la implementación del proveedor de estado. El proveedor de estado predeterminado para la configuración `Persisted` tiene este comportamiento.

El estado se puede recuperar mediante una operación *Get* estándar que inicia `KeyNotFoundException` (C#) o `NoSuchElementException` (Java) si no existe ninguna entrada para la clave:

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public MyActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public Task<int> GetCountAsync()
    {
        return this.StateManager.GetStateAsync<int>("MyState");
    }
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
class MyActorImpl extends FabricActor implements  MyActor
{
    public MyActorImpl(ActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    public CompletableFuture<Integer> getCountAsync()
    {
        return this.stateManager().getStateAsync("MyState");
    }
}
```

El estado también se puede recuperar mediante un método *TryGet* que no se inicia si no existe ninguna entrada para una clave:

```csharp
class MyActor : Actor, IMyActor
{
    public MyActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public async Task<int> GetCountAsync()
    {
        ConditionalValue<int> result = await this.StateManager.TryGetStateAsync<int>("MyState");
        if (result.HasValue)
        {
            return result.Value;
        }

        return 0;
    }
}
```
```Java
class MyActorImpl extends FabricActor implements  MyActor
{
    public MyActorImpl(ActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    public CompletableFuture<Integer> getCountAsync()
    {
        return this.stateManager().<Integer>tryGetStateAsync("MyState").thenApply(result -> {
            if (result.hasValue()) {
                return result.getValue();
            } else {
                return 0;
            });
    }
}
```

## <a name="save-state"></a>Guardar estado
Los métodos de recuperación del administrador de estado devuelven una referencia a un objeto en la memoria local. La modificación de este objeto solo en la memoria local no hace que se guarde de forma duradera. Cuando un objeto se recupera desde el administrador de estado y se modifica, debe volver a insertarse en el administrador de estado para que se guarde de forma duradera.

El estado se puede insertar mediante una operación *Set* no condicional, que equivale a la sintaxis `dictionary["key"] = value`:

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public MyActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public Task SetCountAsync(int value)
    {
        return this.StateManager.SetStateAsync<int>("MyState", value);
    }
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
class MyActorImpl extends FabricActor implements  MyActor
{
    public MyActorImpl(ActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    public CompletableFuture setCountAsync(int value)
    {
        return this.stateManager().setStateAsync("MyState", value);
    }
}
```

Puede agregar el estado mediante un método *Add*. Este método produce `InvalidOperationException` (C#) o `IllegalStateException` (Java) cuando intenta agregar una clave que ya existe.

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public MyActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public Task AddCountAsync(int value)
    {
        return this.StateManager.AddStateAsync<int>("MyState", value);
    }
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
class MyActorImpl extends FabricActor implements  MyActor
{
    public MyActorImpl(ActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    public CompletableFuture addCountAsync(int value)
    {
        return this.stateManager().addOrUpdateStateAsync("MyState", value, (key, old_value) -> old_value + value);
    }
}
```

También puede agregar el estado mediante un método *TryAdd*. Este método no se inicia cuando intenta agregar una clave que ya existe.

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public MyActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public async Task AddCountAsync(int value)
    {
        bool result = await this.StateManager.TryAddStateAsync<int>("MyState", value);

        if (result)
        {
            // Added successfully!
        }
    }
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
class MyActorImpl extends FabricActor implements  MyActor
{
    public MyActorImpl(ActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    public CompletableFuture addCountAsync(int value)
    {
        return this.stateManager().tryAddStateAsync("MyState", value).thenApply((result)->{
            if(result)
            {
                // Added successfully!
            }
        });
    }
}
```

Al final de un método de actor, el administrador de estado guarda automáticamente los valores agregados o modificados por una operación de inserción o actualización. El "guardado" puede incluir la persistencia en el disco y la replicación, dependiendo de la configuración utilizada. Los valores que no se han modificado no se conservan ni se replican. Si no se ha modificado ningún valor, la operación de guardar no hace nada. Si se produce un error al guardar, el estado modificado se descarta y se vuelve a cargar el estado original.

También puede guardar el estado manualmente con una llamada al método `SaveStateAsync` en la base de actor:

```csharp
async Task IMyActor.SetCountAsync(int count)
{
    await this.StateManager.AddOrUpdateStateAsync("count", count, (key, value) => count > value ? count : value);

    await this.SaveStateAsync();
}
```
```Java
interface MyActor {
    CompletableFuture setCountAsync(int count)
    {
        this.stateManager().addOrUpdateStateAsync("count", count, (key, value) -> count > value ? count : value).thenApply();

        this.stateManager().saveStateAsync().thenApply();
    }
}
```

## <a name="remove-state"></a>Quitar estado
Puede quitar el estado de manera permanente de administrador de estado del actor mediante una llamada al método *Remove*. Este método produce `KeyNotFoundException` (C#) o `NoSuchElementException` (Java) cuando intenta quitar una clave que no existe.

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public MyActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public Task RemoveCountAsync()
    {
        return this.StateManager.RemoveStateAsync("MyState");
    }
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
class MyActorImpl extends FabricActor implements  MyActor
{
    public MyActorImpl(ActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    public CompletableFuture removeCountAsync()
    {
        return this.stateManager().removeStateAsync("MyState");
    }
}
```

También puede quitar el estado de forma permanente con el método *TryRemove*. Este método no se inicia cuando intenta quitar una clave que no existe.

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public MyActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public async Task RemoveCountAsync()
    {
        bool result = await this.StateManager.TryRemoveStateAsync("MyState");

        if (result)
        {
            // State removed!
        }
    }
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
class MyActorImpl extends FabricActor implements  MyActor
{
    public MyActorImpl(ActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    public CompletableFuture removeCountAsync()
    {
        return this.stateManager().tryRemoveStateAsync("MyState").thenApply((result)->{
            if(result)
            {
                // State removed!
            }
        });
    }
}
```

## <a name="next-steps"></a>Pasos siguientes

Un estado que se almacena en Reliable Actors debe serializarse antes de escribirse en el disco y replicarse para lograr alta disponibilidad. Aprenda más sobre la [serialización del tipo de actor](service-fabric-reliable-actors-notes-on-actor-type-serialization.md).

A continuación, aprenda más en [Supervisión del rendimiento y diagnósticos de los actores](service-fabric-reliable-actors-diagnostics.md).
