---
title: Eventos en actores de Azure Service Fabric basados en actores
description: Obtenga información sobre los eventos de Reliable Actors de Service Fabric, una forma eficaz de comunicación entre el actor y el cliente.
author: vturecek
ms.topic: conceptual
ms.date: 10/06/2017
ms.author: amanbha
ms.custom: devx-track-csharp
ms.openlocfilehash: f5634a33dccb06437f2e5f095e7880221dba9d6e
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89007917"
---
# <a name="actor-events"></a>Eventos de actor
Los eventos de actor ofrecen una manera de enviar notificaciones de mejor esfuerzo del actor a los clientes. Los eventos de actor están diseñados para la comunicación entre actor y cliente, y no deben usarse para una comunicación entre actores.

Los fragmentos de código siguientes muestran cómo usar los eventos de actor en una aplicación.

Defina una interfaz que describa los eventos publicados por el actor. Esta interfaz debe derivarse de la interfaz `IActorEvents` . Los argumentos de los métodos deben ser [serializable de contratos de datos](service-fabric-reliable-actors-notes-on-actor-type-serialization.md). Los métodos deben devolver void, ya que las notificaciones de eventos son unidireccionales y de mejor esfuerzo.

```csharp
public interface IGameEvents : IActorEvents
{
    void GameScoreUpdated(Guid gameId, string currentScore);
}
```
```Java
public interface GameEvents implements ActorEvents
{
    void gameScoreUpdated(UUID gameId, String currentScore);
}
```
Declare los eventos publicados por el actor en la interfaz del actor.

```csharp
public interface IGameActor : IActor, IActorEventPublisher<IGameEvents>
{
    Task UpdateGameStatus(GameStatus status);

    Task<string> GetGameScore();
}
```
```Java
public interface GameActor extends Actor, ActorEventPublisherE<GameEvents>
{
    CompletableFuture<?> updateGameStatus(GameStatus status);

    CompletableFuture<String> getGameScore();
}
```
En el lado cliente, implemente el controlador de eventos.

```csharp
class GameEventsHandler : IGameEvents
{
    public void GameScoreUpdated(Guid gameId, string currentScore)
    {
        Console.WriteLine(@"Updates: Game: {0}, Score: {1}", gameId, currentScore);
    }
}
```

```Java
class GameEventsHandler implements GameEvents {
    public void gameScoreUpdated(UUID gameId, String currentScore)
    {
        System.out.println("Updates: Game: "+gameId+" ,Score: "+currentScore);
    }
}
```

En el cliente, cree un proxy para el actor que publica el evento y suscríbase a sus eventos.

```csharp
var proxy = ActorProxy.Create<IGameActor>(
                    new ActorId(Guid.Parse(arg)), ApplicationName);

await proxy.SubscribeAsync<IGameEvents>(new GameEventsHandler());
```

```Java
GameActor actorProxy = ActorProxyBase.create<GameActor>(GameActor.class, new ActorId(UUID.fromString(args)));

return ActorProxyEventUtility.subscribeAsync(actorProxy, new GameEventsHandler());
```

Si se producen conmutaciones por error, el actor puede realizar una conmutación por error a un nodo o proceso diferentes. El proxy del actor administra las suscripciones activas y las vuelve a suscribir automáticamente. Puede controlar el intervalo de suscribir nuevamente mediante la API `ActorProxyEventExtensions.SubscribeAsync<TEvent>` . Para cancelar la suscripción, use la API `ActorProxyEventExtensions.UnsubscribeAsync<TEvent>` .

En el actor, publique los eventos cuando se produzcan. Si hay suscriptores al evento, el tiempo de ejecución de los actores les enviará la notificación.

```csharp
var ev = GetEvent<IGameEvents>();
ev.GameScoreUpdated(Id.GetGuidId(), score);
```
```Java
GameEvents event = getEvent<GameEvents>(GameEvents.class);
event.gameScoreUpdated(Id.getUUIDId(), score);
```


## <a name="next-steps"></a>Pasos siguientes
* [Reentrada de actor](service-fabric-reliable-actors-reentrancy.md)
* [Supervisión del rendimiento y diagnósticos de los actores](service-fabric-reliable-actors-diagnostics.md)
* [Documentación de referencia de la API de actor](/previous-versions/azure/dn971626(v=azure.100))
* [Código de ejemplo de C#](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started)
* [Código de ejemplo de .NET Core de C#](https://github.com/Azure-Samples/service-fabric-dotnet-core-getting-started)
* [Código de ejemplo de Java](https://github.com/Azure-Samples/service-fabric-java-getting-started)
