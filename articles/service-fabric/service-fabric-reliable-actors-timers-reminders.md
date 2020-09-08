---
title: Reliable Actors temporizadores y recordatorios
description: Introducción a los temporizadores y avisos de Service Fabric Reliable Actors, incluidas instrucciones sobre cuándo usar cada uno.
ms.topic: conceptual
ms.date: 11/02/2017
ms.custom: devx-track-csharp
ms.openlocfilehash: 2b97b15ca4eb287f8d8f2c1af932f22acafae546
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89016553"
---
# <a name="actor-timers-and-reminders"></a>Recordatorios y temporizadores de los actores
Los actores pueden programar el trabajo periódico mediante el registro de temporizadores o recordatorios. En este artículo se muestra cómo utilizar temporizadores y recordatorios. Además, se explican las diferencias entre ellos.

## <a name="actor-timers"></a>Temporizadores de actor
Los temporizadores de actor ofrecen un contenedor simple alrededor de los temporizadores de .NET o Java para que los métodos de devolución de llamada respeten las garantías de simultaneidad basadas en turnos proporcionados por el tiempo de ejecución de los actores.

Los actores pueden usar los métodos `RegisterTimer`(C#) o `registerTimer`(Java) y `UnregisterTimer`(C#) o `unregisterTimer`(Java) en su clase base para registrar y anular el registro de los temporizadores. En el ejemplo siguiente se muestra el uso de las API de temporizador. Las API son muy similares al temporizador de .NET o de Java. En este ejemplo, cuando el temporizador vence, el entorno de ejecución de los actores llamará al método `MoveObject`(C#) o `moveObject`(Java). Se garantiza que el método respetará la simultaneidad basada en turnos. Esto significa que no habrá otros métodos de actor o devoluciones de llamada de temporizador o recordatorio en curso hasta que esta devolución de llamada complete la ejecución.

```csharp
class VisualObjectActor : Actor, IVisualObject
{
    private IActorTimer _updateTimer;

    public VisualObjectActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    protected override Task OnActivateAsync()
    {
        ...

        _updateTimer = RegisterTimer(
            MoveObject,                     // Callback method
            null,                           // Parameter to pass to the callback method
            TimeSpan.FromMilliseconds(15),  // Amount of time to delay before the callback is invoked
            TimeSpan.FromMilliseconds(15)); // Time interval between invocations of the callback method

        return base.OnActivateAsync();
    }

    protected override Task OnDeactivateAsync()
    {
        if (_updateTimer != null)
        {
            UnregisterTimer(_updateTimer);
        }

        return base.OnDeactivateAsync();
    }

    private Task MoveObject(object state)
    {
        ...
        return Task.FromResult(true);
    }
}
```
```Java
public class VisualObjectActorImpl extends FabricActor implements VisualObjectActor
{
    private ActorTimer updateTimer;

    public VisualObjectActorImpl(FabricActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    @Override
    protected CompletableFuture onActivateAsync()
    {
        ...

        return this.stateManager()
                .getOrAddStateAsync(
                        stateName,
                        VisualObject.createRandom(
                                this.getId().toString(),
                                new Random(this.getId().toString().hashCode())))
                .thenApply((r) -> {
                    this.registerTimer(
                            (o) -> this.moveObject(o),                        // Callback method
                            "moveObject",
                            null,                                             // Parameter to pass to the callback method
                            Duration.ofMillis(10),                            // Amount of time to delay before the callback is invoked
                            Duration.ofMillis(timerIntervalInMilliSeconds));  // Time interval between invocations of the callback method
                    return null;
                });
    }

    @Override
    protected CompletableFuture onDeactivateAsync()
    {
        if (updateTimer != null)
        {
            unregisterTimer(updateTimer);
        }

        return super.onDeactivateAsync();
    }

    private CompletableFuture moveObject(Object state)
    {
        ...
        return this.stateManager().getStateAsync(this.stateName).thenCompose(v -> {
            VisualObject v1 = (VisualObject)v;
            v1.move();
            return (CompletableFuture<?>)this.stateManager().setStateAsync(stateName, v1).
                    thenApply(r -> {
                      ...
                      return null;});
        });
    }
}
```

El siguiente período del temporizador comienza cuando la devolución de llamada completa la ejecución. Esto implica que el temporizador está detenido mientras se ejecuta la devolución de llamada y se inicia cuando esta se completa.

El runtime de los actores guarda los cambios realizados en el administrador de estados del actor cuando finaliza la devolución de llamada. Si se produce un error al guardar el estado, se desactivará dicho objeto de actor y se activará una nueva instancia.

A diferencia de los [recordatorios](#actor-reminders), los temporizadores no se pueden actualizar. Si se llama de nuevo a `RegisterTimer`, se registrará un nuevo temporizador.

Todos los temporizadores se detienen cuando el actor se desactiva como parte de la recolección de elementos no utilizados. Después de eso, no se invoca ninguna devolución de llamada de temporizador. Además, el tiempo de ejecución de los actores no conserva ninguna información sobre los temporizadores que se estaban ejecutando antes de la desactivación. Es el actor el que decide si se registran los temporizadores que necesita cuando se reactive en el futuro. Para obtener más información, consulte la sección [Recolección de actores no utilizados](service-fabric-reliable-actors-lifecycle.md).

## <a name="actor-reminders"></a>Recordatorios de actor
Los recordatorios son un mecanismo para desencadenar devoluciones de llamada persistentes en un actor en momentos especificados. Su funcionalidad es similar a la de los temporizadores. Sin embargo, a diferencia de los temporizadores, los recordatorios se desencadenan en todas las circunstancias hasta que el actor las registre o elimine del registro expresamente. En concreto, los recordatorios se activan en conmutaciones por error y desactivaciones de actores, ya que el tiempo de ejecución de los actores conserva la información sobre los recordatorios de actor mediante un proveedor de estado. Además, a diferencia de los temporizadores, los recordatorios existentes se pueden actualizar llamando al método de registro (`RegisterReminderAsync`) de nuevo con el mismo *reminderName*.

> [!NOTE]
> La confiabilidad de los recordatorios está ligada a las garantías de confiabilidad del estado proporcionadas por el proveedor de estado del actor. Esto significa que para los actores cuya persistencia de estado se establece en *Ninguna*, los recordatorios no se activarán después de una conmutación por error.

Para registrar un recordatorio, un actor llama al método [`RegisterReminderAsync`](/dotnet/api/microsoft.servicefabric.actors.runtime.actorbase.registerreminderasync?view=azure-dotnet#remarks) que se ofrece en la clase base, tal y como se muestra en el ejemplo siguiente:

```csharp
protected override async Task OnActivateAsync()
{
    string reminderName = "Pay cell phone bill";
    int amountInDollars = 100;

    IActorReminder reminderRegistration = await this.RegisterReminderAsync(
        reminderName,
        BitConverter.GetBytes(amountInDollars),
        TimeSpan.FromDays(3),    //The amount of time to delay before firing the reminder
        TimeSpan.FromDays(1));    //The time interval between firing of reminders
}
```

```Java
@Override
protected CompletableFuture onActivateAsync()
{
    String reminderName = "Pay cell phone bill";
    int amountInDollars = 100;

    ActorReminder reminderRegistration = this.registerReminderAsync(
            reminderName,
            state,
            dueTime,    //The amount of time to delay before firing the reminder
            period);    //The time interval between firing of reminders
}
```

En este ejemplo, `"Pay cell phone bill"` es el nombre del recordatorio. Se trata de una cadena que el actor usa para identificar un recordatorio de forma única. `BitConverter.GetBytes(amountInDollars)`(C#) es el contexto que está asociado con el recordatorio. Se pasará al actor como un argumento en la devolución de llamada del recordatorio, es decir, `IRemindable.ReceiveReminderAsync`(C#) o `Remindable.receiveReminderAsync`(Java).

Los actores que usan recordatorios deben implementar la interfaz `IRemindable` , tal y como se muestra en el ejemplo siguiente.

```csharp
public class ToDoListActor : Actor, IToDoListActor, IRemindable
{
    public ToDoListActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public Task ReceiveReminderAsync(string reminderName, byte[] context, TimeSpan dueTime, TimeSpan period)
    {
        if (reminderName.Equals("Pay cell phone bill"))
        {
            int amountToPay = BitConverter.ToInt32(context, 0);
            System.Console.WriteLine("Please pay your cell phone bill of ${0}!", amountToPay);
        }
        return Task.FromResult(true);
    }
}
```
```Java
public class ToDoListActorImpl extends FabricActor implements ToDoListActor, Remindable
{
    public ToDoListActor(FabricActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    public CompletableFuture receiveReminderAsync(String reminderName, byte[] context, Duration dueTime, Duration period)
    {
        if (reminderName.equals("Pay cell phone bill"))
        {
            int amountToPay = ByteBuffer.wrap(context).getInt();
            System.out.println("Please pay your cell phone bill of " + amountToPay);
        }
        return CompletableFuture.completedFuture(true);
    }

```

Cuando un recordatorio se desencadena, el entorno de ejecución de Reliable Actors invocará al método `ReceiveReminderAsync`(C#) o `receiveReminderAsync`(Java) en el actor. Un actor puede registrar varios recordatorios y el método `ReceiveReminderAsync`(C#) o `receiveReminderAsync`(Java) se invoca cuando se desencadene uno de ellos. El actor puede usar el nombre del recordatorio que se pasa al método `ReceiveReminderAsync`(C#) o `receiveReminderAsync`(Java) para averiguar el recordatorio que se activó.

El entorno de ejecución de Actors guarda el estado del actor cuando termina la llamada a `ReceiveReminderAsync`(C#) o `receiveReminderAsync`(Java). Si se produce un error al guardar el estado, se desactivará dicho objeto de actor y se activará una nueva instancia.

Para anular el registro de un recordatorio, un actor llama al método `UnregisterReminderAsync`(C#) o `unregisterReminderAsync`(Java), tal y como se muestra en el ejemplo siguiente.

```csharp
IActorReminder reminder = GetReminder("Pay cell phone bill");
Task reminderUnregistration = UnregisterReminderAsync(reminder);
```
```Java
ActorReminder reminder = getReminder("Pay cell phone bill");
CompletableFuture reminderUnregistration = unregisterReminderAsync(reminder);
```

Como se indicó anteriormente, el método `UnregisterReminderAsync`(C#) o `unregisterReminderAsync`(Java) acepta una interfaz `IActorReminder`(C#) o `ActorReminder`(Java). La clase base del actor admite un método `GetReminder`(C#) o `getReminder`(Java) que puede usarse para recuperar la interfaz `IActorReminder`(C#) o `ActorReminder`(Java) pasándole el nombre del recordatorio. Esto resulta útil porque el actor no necesita conservar la interfaz `IActorReminder`(C#) o `ActorReminder`(Java) que devuelve la llamada al método `RegisterReminder`(C#) o `registerReminder`(Java).

## <a name="next-steps"></a>Pasos siguientes
Obtenga información sobre los eventos y la reentrada de Reliable Actor:
* [Eventos de actor](service-fabric-reliable-actors-events.md)
* [Reentrada de actor](service-fabric-reliable-actors-reentrancy.md)
