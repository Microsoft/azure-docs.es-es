---
title: Reliable Actors en Service Fabric
description: Describe cómo Reliable Actors se superpone encima de Reliable Services y usa las características de la plataforma Service Fabric.
author: vturecek
ms.topic: conceptual
ms.date: 3/9/2018
ms.author: vturecek
ms.custom: devx-track-csharp
ms.openlocfilehash: 1cd90d4567bde6cd2c4f2a29e2d516b51b79e2af
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89016621"
---
# <a name="how-reliable-actors-use-the-service-fabric-platform"></a>Uso de la plataforma Service Fabric por parte de actores confiables
En este artículo se explica cómo funciona Reliable Actors en la plataforma de Azure Service Fabric. Reliable Actors se ejecuta en un marco de trabajo que se hospeda en la implementación de un servicio de confianza con estado llamado *servicio de actor*. El servicio de actor contiene todos los componentes necesarios para administrar el ciclo de vida y el envío de mensajes de sus actores:

* El tiempo de ejecución de los actores administra el ciclo de vida y la recolección de elementos no utilizados, además de aplicar el acceso uniproceso.
* Un servicio de actor que actúa como agente de escucha remoto acepta llamadas de acceso remoto a los actores y las envía a un distribuidor que las redirige a la instancia de actor correspondiente.
* El proveedor de estado de actores encapsula proveedores de estado (como el proveedor de estado de Reliable Collections) y proporciona un adaptador para la administración de estados de los actores.

En conjunto, estos componentes forman el marco de trabajo de Reliable Actor.

## <a name="service-layering"></a>Servicio en capas
Debido a que el propio servicio de actor es de tipo Reliable Services, todos los conceptos de [modelo de aplicación](service-fabric-application-model.md), ciclo de vida, [empaquetado](service-fabric-package-apps.md), [implementación](service-fabric-deploy-remove-applications.md), actualización y escalado de Reliable Services se aplican del mismo modo a los servicios de actor.

![Servicio de actor en capas][1]

El diagrama anterior muestra la relación que existe entre los marcos de trabajo de la aplicación Service Fabric y el código de usuario. Los elementos en azul representan el marco de trabajo de la aplicación Reliable Services, el color naranjo representa el marco de trabajo de Reliable Actor y el verde, el código de usuario.

En Reliable Services, el servicio hereda la clase `StatefulService`. Esta clase deriva de `StatefulServiceBase`, o bien de `StatelessService` en el caso de los servicios sin estado. En Reliable Actors, use el servicio de actor. El servicio de actor es una implementación diferente de la clase `StatefulServiceBase` que implementa el patrón de actor donde se ejecutan los actores. Debido a que el servicio de actor es simplemente una implementación de `StatefulServiceBase`, puede escribir su propio servicio que deriva de `ActorService` e implementar características de nivel de servicio del mismo modo que lo haría si hereda `StatefulService`, como:

* Copia de seguridad y restauración del servicio.
* Funcionalidad compartida para todos los actores, por ejemplo, un interruptor.
* Llamadas de procedimiento remoto en el propio servicio de actor y en cada actor individual.

Para obtener más información, consulte [Implementing service-level features in your actor service](service-fabric-reliable-actors-using.md) (Implementación de características de nivel de servicio en el servicio de actor).

## <a name="application-model"></a>Modelo de aplicación
Los servicios de actor son Reliable Services, por lo que el modelo de aplicación es el mismo. Sin embargo, las herramientas de generación del marco de trabajo de actor generan algunos de los archivos del modelo de aplicación por usted.

### <a name="service-manifest"></a>Manifiesto de servicio
Las herramientas de generación del marco de trabajo de actor generan automáticamente el contenido del archivo ServiceManifest.xml del servicio de actor. Este archivo incluye:

* El tipo de servicio de actor. El nombre de tipo se genera en función del nombre del proyecto de actor. La marca HasPersistedState se define según el atributo de persistencia del actor.
* Paquete de código.
* Paquete de configuración.
* Recursos y puntos de conexión.

### <a name="application-manifest"></a>Manifiesto de aplicación
Las herramientas de generación del marco de trabajo de actor crea automáticamente una definición de servicio predeterminada para el servicio de actor. Las herramientas de generación rellenan las propiedades predeterminadas de servicio:

* El atributo de persistencia del actor determina la cantidad de conjuntos de réplicas. Cada vez que cambia el atributo de persistencia del actor, se restablece también la cantidad de conjuntos de réplicas en la definición de servicio predeterminada.
* El intervalo y el esquema de partición se definen en Int64 uniforme con todo el intervalo de claves Int64.

## <a name="service-fabric-partition-concepts-for-actors"></a>Conceptos de partición de Service Fabric para los actores
Los servicios de actor son servicios con estado particionados. Cada partición de un servicio de actor contiene un conjunto de actores. Las particiones de servicio se distribuyen automáticamente sobre varios nodos en Service Fabric. Como resultado, se distribuyen las instancias de actor.

![Partición y distribución de actores][5]

Es posible crear Reliable Services con distintos intervalos de claves de partición y esquemas de partición. El servicio de actor usa el esquema de partición Int64 con todo el intervalo de claves Int64 para asignar actores a las particiones.

### <a name="actor-id"></a>Id. de actor
Cada actor que se crea en el servicio tiene asociado un id. único, el que se representa con la clase `ActorId`. La clase `ActorId` es un valor de identificador opaco que se puede usar para la distribución uniforme de actores en todas las particiones de servicio a través de la generación de identificadores aleatorios:

```csharp
ActorProxy.Create<IMyActor>(ActorId.CreateRandom());
```
```Java
ActorProxyBase.create<MyActor>(MyActor.class, ActorId.newId());
```


A cada `ActorId` se aplica un algoritmo hash en Int64. Es el motivo por el cual el servicio de actor debe usar un esquema de partición Int64 con todo el intervalo de claves Int64. Sin embargo, los valores de identificadores personalizados se pueden usar para un valor `ActorID`, incluidos GUID/UUID, cadenas y valores Int64.

```csharp
ActorProxy.Create<IMyActor>(new ActorId(Guid.NewGuid()));
ActorProxy.Create<IMyActor>(new ActorId("myActorId"));
ActorProxy.Create<IMyActor>(new ActorId(1234));
```
```Java
ActorProxyBase.create(MyActor.class, new ActorId(UUID.randomUUID()));
ActorProxyBase.create(MyActor.class, new ActorId("myActorId"));
ActorProxyBase.create(MyActor.class, new ActorId(1234));
```

Cuando se usan GUID/UUID y cadenas, se aplica un algoritmo hash a los valores en un Int64. Sin embargo, cuando se brinda explícitamente un valor Int64 a una clase `ActorId`, el valor Int64 se asignará directamente a una partición sin otra aplicación del algoritmo hash. Puede usar esta técnica para controlar en qué partición se ubican los actores.


## <a name="next-steps"></a>Pasos siguientes
* [Administración de estados de los actores](service-fabric-reliable-actors-state-management.md)
* [Ciclo de vida de un actor y recolección de elementos no utilizados](service-fabric-reliable-actors-lifecycle.md)
* [Documentación de referencia de la API de actores](/dotnet/api/microsoft.servicefabric.actors?view=azure-dotnet)
* [Código de ejemplo de .NET](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started)
* [Código de ejemplo de Java](https://github.com/Azure-Samples/service-fabric-java-getting-started)

<!--Image references-->
[1]: ./media/service-fabric-reliable-actors-platform/actor-service.png
[2]: ./media/service-fabric-reliable-actors-platform/app-deployment-scripts.png
[3]: ./media/service-fabric-reliable-actors-platform/actor-partition-info.png
[4]: ./media/service-fabric-reliable-actors-platform/actor-replica-role.png
[5]: ./media/service-fabric-reliable-actors-introduction/distribution.png
