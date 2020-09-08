---
title: Reentrada de actores en Azure Service Fabric
description: Introducción a la reentrada Reliable Actors de Azure Service Fabric, una manera de evitar mediante lógica el bloqueo según el contexto de llamada.
author: vturecek
ms.topic: conceptual
ms.date: 11/02/2017
ms.author: vturecek
ms.custom: devx-track-csharp
ms.openlocfilehash: 75512633dae8d9a9f61c37bbc795d8e8d20c4ff1
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89007815"
---
# <a name="reliable-actors-reentrancy"></a>Reentrada de Reliable Actors
El runtime de Reliable Actors permite, de manera predeterminada, la reentrada basada en el contexto de llamadas lógicas. Esto permite que los actores sean reentrantes si están en la misma cadena de contexto de llamada. Por ejemplo, si el actor A envía un mensaje al actor B, quien envía el mensaje al actor C; como parte del procesamiento del mensaje, si el actor C llama al actor A, el mensaje es reentrante y, por los tanto, se permitirá. Los demás mensajes que formen parte de un contexto de llamada distinto se bloquearán en el actor A hasta que complete el procesamiento.

Existen dos opciones disponibles para la reentrada de actores definidas en la enumeración `ActorReentrancyMode` :

* `LogicalCallContext` (comportamiento predeterminado)
* `Disallowed` : deshabilita la reentrada.

```csharp
public enum ActorReentrancyMode
{
    LogicalCallContext = 1,
    Disallowed = 2
}
```
```Java
public enum ActorReentrancyMode
{
    LogicalCallContext(1),
    Disallowed(2)
}
```
Se puede establecer la reentrada en la configuración de una clase `ActorService`durante el registro. La configuración se aplica a todas las instancias de actor que creó en el servicio de actor.

En el ejemplo siguiente se muestra un servicio de actor que establece el modo de reentrada en `ActorReentrancyMode.Disallowed`. En este caso, si un actor envía un mensaje reentrante a otro actor, se generará una excepción de tipo `FabricException` .

```csharp
static class Program
{
    static void Main()
    {
        try
        {
            ActorRuntime.RegisterActorAsync<Actor1>(
                (context, actorType) => new ActorService(
                    context,
                    actorType, () => new Actor1(),
                    settings: new ActorServiceSettings()
                    {
                        ActorConcurrencySettings = new ActorConcurrencySettings()
                        {
                            ReentrancyMode = ActorReentrancyMode.Disallowed
                        }
                    }))
                .GetAwaiter().GetResult();

            Thread.Sleep(Timeout.Infinite);
        }
        catch (Exception e)
        {
            ActorEventSource.Current.ActorHostInitializationFailed(e.ToString());
            throw;
        }
    }
}
```
```Java
static class Program
{
    static void Main()
    {
        try
        {
            ActorConcurrencySettings actorConcurrencySettings = new ActorConcurrencySettings();
            actorConcurrencySettings.setReentrancyMode(ActorReentrancyMode.Disallowed);

            ActorServiceSettings actorServiceSettings = new ActorServiceSettings();
            actorServiceSettings.setActorConcurrencySettings(actorConcurrencySettings);

            ActorRuntime.registerActorAsync(
                Actor1.getClass(),
                (context, actorType) -> new FabricActorService(
                    context,
                    actorType, () -> new Actor1(),
                    null,
                    stateProvider,
                    actorServiceSettings, timeout);

            Thread.sleep(Long.MAX_VALUE);
        }
        catch (Exception e)
        {
            throw e;
        }
    }
}
```


## <a name="next-steps"></a>Pasos siguientes
* Aprenda más sobre la reentrada en la [documentación de referencia de Actor API](/previous-versions/azure/dn971626(v=azure.100))
