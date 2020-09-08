---
title: Polimorfismo en el marco de trabajo de Reliable Actors
description: Cree jerarquías de tipos e interfaces de .NET en el marco de trabajo de Reliable Actors para volver a usar la funcionalidad y las definiciones de la API.
author: vturecek
ms.topic: conceptual
ms.date: 11/02/2017
ms.author: vturecek
ms.custom: devx-track-csharp
ms.openlocfilehash: 7552609fc4c85cdd5033e67b1bab7fb58cb6ebf1
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89016570"
---
# <a name="polymorphism-in-the-reliable-actors-framework"></a>Polimorfismo en el marco de trabajo de Reliable Actors
El marco de Reliable Actors permite compilar actores empleando muchas de las mismas técnicas que usaría en el diseño orientado a objetos. Una de esas técnicas es el polimorfismo, que los tipos y las interfaces hereden de elementos primarios más generalizados. La herencia en el marco de Reliable Actors suele seguir el modelo de .NET con algunas restricciones adicionales. En el caso de Java/Linux, sigue el modelo de Java.

## <a name="interfaces"></a>Interfaces
El marco Reliable Actors requiere la definición de al menos una interfaz que se implementará por el tipo de actor. Esta interfaz se usa para generar una clase de proxy que se puede usar por los clientes para comunicarse con los actores. Las interfaces pueden heredar elementos de otras interfaces, siempre y cuando todas las implementadas por un tipo de actor y todos sus elementos principales deriven de IActor(C#) o Actor(Java) en última instancia. IActor(C#) y Actor(Java) son las interfaces base definidas por plataforma destinadas a actores en .NET Framework y en la plataforma Java, respectivamente. Así, el ejemplo clásico de polimorfismo en el que se usan formas sería algo parecido a lo siguiente:

![Jerarquía de la interfaz de los actores de forma][shapes-interface-hierarchy]

## <a name="types"></a>Tipos
También puede crear una jerarquía de tipos de actores, que se derivan de la clase Actor base proporcionada por la plataforma. En el caso de las formas, es posible tener un tipo base `Shape`(C#) o `ShapeImpl`(Java):

```csharp
public abstract class Shape : Actor, IShape
{
    public abstract Task<int> GetVerticeCount();

    public abstract Task<double> GetAreaAsync();
}
```
```Java
public abstract class ShapeImpl extends FabricActor implements Shape
{
    public abstract CompletableFuture<int> getVerticeCount();

    public abstract CompletableFuture<double> getAreaAsync();
}
```

Los subtipos de `Shape`(C#) o `ShapeImpl`(Java) pueden invalidar métodos del tipo base.

```csharp
[ActorService(Name = "Circle")]
[StatePersistence(StatePersistence.Persisted)]
public class Circle : Shape, ICircle
{
    public override Task<int> GetVerticeCount()
    {
        return Task.FromResult(0);
    }

    public override async Task<double> GetAreaAsync()
    {
        CircleState state = await this.StateManager.GetStateAsync<CircleState>("circle");

        return Math.PI *
            state.Radius *
            state.Radius;
    }
}
```
```Java
@ActorServiceAttribute(name = "Circle")
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
public class Circle extends ShapeImpl implements Circle
{
    @Override
    public CompletableFuture<Integer> getVerticeCount()
    {
        return CompletableFuture.completedFuture(0);
    }

    @Override
    public CompletableFuture<Double> getAreaAsync()
    {
        return (this.stateManager().getStateAsync<CircleState>("circle").thenApply(state->{
          return Math.PI * state.radius * state.radius;
        }));
    }
}
```

Observe el atributo `ActorService` en el tipo de actor. Este atributo indica al marco de Reliable Actors que debe crear automáticamente un servicio para hospedar actores de este tipo. En algunos casos, quizás quiera crear un tipo base que esté pensado únicamente para compartir la funcionalidad con subtipos y que nunca se usará para crear instancias de actores concretos. En esos casos, debe usar la palabra clave `abstract` para indicar que nunca va a crear un actor basado en ese tipo.

## <a name="next-steps"></a>Pasos siguientes
* Consulte [cómo el marco de trabajo de Reliable Actors aprovecha la plataforma de Service Fabric](service-fabric-reliable-actors-platform.md) para ofrecer confiabilidad, escalabilidad y un estado coherente.
* Obtenga información sobre el [ciclo de vida de los actores](service-fabric-reliable-actors-lifecycle.md).

<!-- Image references -->

[shapes-interface-hierarchy]: ./media/service-fabric-reliable-actors-polymorphism/Shapes-Interface-Hierarchy.png
