---
title: Implementación de características en actores de Azure Service Fabric
description: Se describe cómo escribir su propio servicio de actor que implemente características de nivel de servicio de la misma manera que al heredar StatefulService.
author: vturecek
ms.topic: conceptual
ms.date: 03/19/2018
ms.author: vturecek
ms.custom: devx-track-csharp
ms.openlocfilehash: 4eb8dd98909459feab72f61e46834e60c5b07ebd
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89007764"
---
# <a name="implement-service-level-features-in-your-actor-service"></a>Implementación de características de nivel de servicio en el servicio de actor

Como se describe en las [capas de servicio](service-fabric-reliable-actors-platform.md#service-layering), el propio servicio de actor es un servicio confiable. Puede escribir su propio servicio que se deriva de `ActorService`. También puede implementar características de nivel de servicio de la misma manera que cuando hereda un servicio con estado, como:

- Copia de seguridad y restauración del servicio.
- Funcionalidad compartida para todos los actores, por ejemplo, un interruptor.
- Llamadas de procedimiento remoto en el propio servicio de actor y en cada actor individual.

## <a name="use-the-actor-service"></a>Uso del servicio de actor

Las instancias de actor tienen acceso al servicio de actor en que se ejecutan. A través del servicio de actor, las instancias de actor pueden obtener el contexto de servicio mediante programación. El contexto de servicio tiene el identificador de partición, el nombre del servicio, el nombre de la aplicación y otra información específica de la plataforma de Azure Service Fabric:

```csharp
Task MyActorMethod()
{
    Guid partitionId = this.ActorService.Context.PartitionId;
    string serviceTypeName = this.ActorService.Context.ServiceTypeName;
    Uri serviceInstanceName = this.ActorService.Context.ServiceName;
    string applicationInstanceName = this.ActorService.Context.CodePackageActivationContext.ApplicationName;
}
```
```Java
CompletableFuture<?> MyActorMethod()
{
    UUID partitionId = this.getActorService().getServiceContext().getPartitionId();
    String serviceTypeName = this.getActorService().getServiceContext().getServiceTypeName();
    URI serviceInstanceName = this.getActorService().getServiceContext().getServiceName();
    String applicationInstanceName = this.getActorService().getServiceContext().getCodePackageActivationContext().getApplicationName();
}
```

Del mismo modo que ocurre con todos los Reliable Services, el servicio de actor se debe registrar con un tipo de servicio en el tiempo de ejecución de Service Fabric. Con el fin de que el servicio de actor ejecute las instancias de actor, el tipo de actor también se debe registrar con el servicio de actor. El método de registro `ActorRuntime` realiza este trabajo para los actores. En el caso más sencillo, puede registrar el tipo de actor y entonces el servicio de actor usará la configuración predeterminada.

```csharp
static class Program
{
    private static void Main()
    {
        ActorRuntime.RegisterActorAsync<MyActor>().GetAwaiter().GetResult();

        Thread.Sleep(Timeout.Infinite);
    }
}
```

Como alternativa, puede usar una expresión lambda para construir el servicio de actor. A continuación, puede configurar el servicio de actor y construir de manera explícita las instancias de actor. Puede insertar dependencias en el actor mediante su constructor.

```csharp
static class Program
{
    private static void Main()
    {
        ActorRuntime.RegisterActorAsync<MyActor>(
            (context, actorType) => new ActorService(context, actorType, () => new MyActor()))
            .GetAwaiter().GetResult();

        Thread.Sleep(Timeout.Infinite);
    }
}
```
```Java
static class Program
{
    private static void Main()
    {
      ActorRuntime.registerActorAsync(
              MyActor.class,
              (context, actorTypeInfo) -> new FabricActorService(context, actorTypeInfo),
              timeout);

        Thread.sleep(Long.MAX_VALUE);
    }
}
```

## <a name="actor-service-methods"></a>Métodos de servicio de actor

El servicio de actor implementa `IActorService` (C#) o `ActorService` (Java), que, a su vez, implementa `IService` (C#) o `Service` (Java). Esta es la interfaz que se usa en la comunicación remota de Reliable Services, que permite llamadas a procedimiento remoto en los métodos de servicio. Contiene métodos de nivel de servicio a los que se puede llamar de manera remota a través del servicio remoto. Se puede usar para [enumerar](service-fabric-reliable-actors-enumerate.md) y [eliminar](service-fabric-reliable-actors-delete-actors.md) actores.


## <a name="custom-actor-service"></a>Servicio de actor personalizado

Mediante el uso de la expresión lambda del registro de actor, puede registrar su propio servicio de actor personalizado que deriva de `ActorService` (C#) y `FabricActorService` (Java). Luego puede implementar su propia funcionalidad de nivel de servicio mediante la escritura de una clase de servicio que hereda `ActorService` (C#) o `FabricActorService` (Java). Un servicio de actor personalizado hereda toda la funcionalidad en tiempo de ejecución de actor de `ActorService` (C#) o `FabricActorService` (Java). Se puede usar para implementar sus propios métodos de servicio.

```csharp
class MyActorService : ActorService
{
    public MyActorService(StatefulServiceContext context, ActorTypeInformation typeInfo, Func<ActorBase> newActor)
        : base(context, typeInfo, newActor)
    { }
}
```
```Java
class MyActorService extends FabricActorService
{
    public MyActorService(StatefulServiceContext context, ActorTypeInformation typeInfo, BiFunction<FabricActorService, ActorId, ActorBase> newActor)
    {
         super(context, typeInfo, newActor);
    }
}
```

```csharp
static class Program
{
    private static void Main()
    {
        ActorRuntime.RegisterActorAsync<MyActor>(
            (context, actorType) => new MyActorService(context, actorType, () => new MyActor()))
            .GetAwaiter().GetResult();

        Thread.Sleep(Timeout.Infinite);
    }
}
```
```Java
public class Program
{
    public static void main(String[] args)
    {
        ActorRuntime.registerActorAsync(
                MyActor.class,
                (context, actorTypeInfo) -> new FabricActorService(context, actorTypeInfo),
                timeout);
        Thread.sleep(Long.MAX_VALUE);
    }
}
```

## <a name="implement-actor-backup-and-restore"></a>Implementación de la copia de seguridad y la restauración de actores

Un servicio de actor personalizado puede exponer un método para crear una copia de seguridad de los datos de actor mediante el uso del agente de escucha remoto que ya se encuentra presente en `ActorService`: Para obtener un ejemplo, vea [Actores de copia de seguridad y restauración](service-fabric-reliable-actors-backup-and-restore.md).

## <a name="actor-that-uses-a-remoting-v2-interface-compatible-stack"></a>Actor que usa una pila de comunicación remota V2 (compatible con interfaz)

La pila de comunicación remota V2 (compatible con interfaz, conocida como V2_1) tiene todas las características de la pila de comunicación remota V2. Su interfaz es compatible con la pila de comunicación remota V1, pero no es compatible con V2 ni V1. Para actualizar de V1 a V2_1 sin que la disponibilidad del servicio se vea afectada, siga los pasos descritos en la sección siguiente.

Los cambios siguientes son necesarios para usar la pila de comunicación remota V2_1:

1. Agregue el siguiente atributo de ensamblado en las interfaces de actor.
  
   ```csharp
   [assembly:FabricTransportActorRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2_1,RemotingClientVersion = RemotingClientVersion.V2_1)]
   ```

2. Compile y actualice los proyectos de servicio de actor y cliente de actor para empezar a usar la pila V2.

### <a name="actor-service-upgrade-to-remoting-v2-interface-compatible-stack-without-affecting-service-availability"></a>Actualización del servicio de actor a la pila de comunicación remota V2 (compatible con interface) sin que la disponibilidad del servicio se vea afectada

Este cambio es una actualización en dos pasos. Siga los pasos de esta secuencia.

1. Agregue el siguiente atributo de ensamblado en las interfaces de actor. Este atributo inicia dos agentes de escucha para el servicio de actor, V1 (existente) y el agente de escucha V2_1. Actualice el servicio de actor con este cambio.

   ```csharp
   [assembly:FabricTransportActorRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V1|RemotingListenerVersion.V2_1,RemotingClientVersion = RemotingClientVersion.V2_1)]
   ```

2. Actualice los clientes de actor después de completar la actualización anterior.
   Este paso garantiza que el proxy de actor usa la pila de comunicación remota V2_1.

3. Este paso es opcional. Cambie el atributo anterior para quitar el agente de escucha V1.

    ```csharp
    [assembly:FabricTransportActorRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2_1,RemotingClientVersion = RemotingClientVersion.V2_1)]
    ```

## <a name="actor-that-uses-the-remoting-v2-stack"></a>Actor que usa la pila de comunicación remota V2

Con el paquete NuGet 2.8, los usuarios ahora pueden utilizar la pila de comunicación remota V2, que funciona mejor y proporciona características como serialización personalizada. La comunicación remota V2 no es compatible con versiones anteriores de la pila de comunicación remota existente (llamada ahora la pila de comunicación remota V1).

Para usar la pila de comunicación remota V2, son necesarios los cambios siguientes:

1. Agregue el siguiente atributo de ensamblado en las interfaces de actor.

   ```csharp
   [assembly:FabricTransportActorRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2,RemotingClientVersion = RemotingClientVersion.V2)]
   ```

2. Compile y actualice los proyectos de servicio de actor y cliente de actor para empezar a usar la pila V2.

### <a name="upgrade-the-actor-service-to-the-remoting-v2-stack-without-affecting-service-availability"></a>Actualización del servicio de actor a la pila de comunicación remota V2 sin que la disponibilidad del servicio se vea afectada

Este cambio es una actualización en dos pasos. Siga los pasos de esta secuencia.

1. Agregue el siguiente atributo de ensamblado en las interfaces de actor. Este atributo inicia dos agentes de escucha para el servicio de actor, V1 (existente) y el agente de escucha V2. Actualice el servicio de actor con este cambio.

   ```csharp
   [assembly:FabricTransportActorRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V1|RemotingListenerVersion.V2,RemotingClientVersion = RemotingClientVersion.V2)]
   ```

2. Actualice los clientes de actor después de completar la actualización anterior.
   Este paso garantiza que el proxy de actor usa la pila de comunicación remota V2.

3. Este paso es opcional. Cambie el atributo anterior para quitar el agente de escucha V1.

    ```csharp
    [assembly:FabricTransportActorRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2,RemotingClientVersion = RemotingClientVersion.V2)]
    ```

## <a name="next-steps"></a>Pasos siguientes

* [Administración de estados de los actores](service-fabric-reliable-actors-state-management.md)
* [Ciclo de vida de un actor y recolección de elementos no utilizados](service-fabric-reliable-actors-lifecycle.md)
* [Documentación de referencia de la API de actores](/previous-versions/azure/dn971626(v=azure.100))
* [Código de ejemplo de .NET](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started)
* [Código de ejemplo de Java](https://github.com/Azure-Samples/service-fabric-java-getting-started)

<!--Image references-->
[1]: ./media/service-fabric-reliable-actors-platform/actor-service.png
[2]: ./media/service-fabric-reliable-actors-platform/app-deployment-scripts.png
[3]: ./media/service-fabric-reliable-actors-platform/actor-partition-info.png
[4]: ./media/service-fabric-reliable-actors-platform/actor-replica-role.png
[5]: ./media/service-fabric-reliable-actors-introduction/distribution.png
