---
title: Administración de estado de Reliable Actors
description: Describe cómo se administra, se conserva y se replica el estado de Reliable Actors para alta disponibilidad.
author: vturecek
ms.topic: conceptual
ms.date: 11/02/2017
ms.author: vturecek
ms.custom: devx-track-csharp
ms.openlocfilehash: 92abfa9937c1ac3f7d4ba68e9228b29e0211e1af
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89007798"
---
# <a name="reliable-actors-state-management"></a>Administración de estado de Reliable Actors
Reliable Actors son objetos uniproceso que encapsulan la lógica y el estado. Como los actores se ejecutan en Reliable Services, pueden mantener el estado de forma confiable con los mismos mecanismos de persistencia y replicación. De este modo, los actores no pierden su estado después de los errores, cuando se reactivan después de la recolección de elementos no utilizados o cuando se mueven entre los nodos de un clúster debido un equilibrio de los recursos o actualizaciones.

## <a name="state-persistence-and-replication"></a>Persistencia y replicación del estado
Reliable Actors se considera *con estado* porque cada instancia de actor se asigna a un identificador único. Esto significa que las llamadas repetidas al mismo identificador de actor se enrutan a la misma instancia de actor. En un sistema sin estado, en cambio, no se garantiza que las llamadas del cliente se enruten al mismo servidor cada vez. Por este motivo, los servicios de actor siempre son servicios con estado.

Aunque los actores se consideran con estado, no significa que deben almacenar el estado de manera confiable. Los actores pueden elegir el nivel de persistencia y replicación de estado basado en los requisitos de almacenamiento de sus datos:

* **Estado persistente**: el estado se conserva en el disco y se replica a tres o más réplicas. El estado persistente es la opción de almacenamiento de estado más duradera, ya que el estado puede persistir a través de la interrupción de todo el clúster.
* **Estado volátil**: el estado se replica a tres o más réplicas y solo se conserva en memoria. El estado volátil proporciona resistencia frente a errores de nodo, errores de actores y durante las actualizaciones y el equilibrio de recursos. Sin embargo, el estado no persiste en el disco. Por tanto, si se pierden todas las réplicas al mismo tiempo, el estado también se pierde.
* **Sin estado persistente**: el estado no se replica ni se escribe en el disco. Solo se usa para actores que no necesitan mantener el estado de forma confiable.

Cada nivel de persistencia es simplemente otra configuración del *proveedor de estado* y de la *replicación* del servicio. Si el estado se escribe o no en el disco depende del proveedor de estado (el componente de un servicio Reliable Services que almacena el estado). La replicación depende de con cuántas réplicas se ha implementado un servicio. Al igual que con Reliable Services, tanto el proveedor de estado como el número de réplicas pueden establecerse manualmente con facilidad. La plataforma de actores proporciona un atributo, que, cuando se utiliza en un actor, selecciona automáticamente un proveedor de estado predeterminado y genera automáticamente la configuración para el número de réplicas, a fin de lograr una de estas tres opciones de persistencia. El atributo StatePersistence no es heredado por una clase derivada; cada tipo de actor debe proporcionar su nivel de StatePersistence.

### <a name="persisted-state"></a>Estado persistente
```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
class MyActorImpl  extends FabricActor implements MyActor
{
}
```  
Esta configuración usa un proveedor de estado que almacena datos en el disco y establece automáticamente el número de réplicas del servicio en 3.

### <a name="volatile-state"></a>Estado volátil
```csharp
[StatePersistence(StatePersistence.Volatile)]
class MyActor : Actor, IMyActor
{
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Volatile)
class MyActorImpl extends FabricActor implements MyActor
{
}
```
Esta configuración utiliza un proveedor de estado solo en memoria y establece el número de réplicas en 3.

### <a name="no-persisted-state"></a>Sin estado persistente
```csharp
[StatePersistence(StatePersistence.None)]
class MyActor : Actor, IMyActor
{
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.None)
class MyActorImpl extends FabricActor implements MyActor
{
}
```
Esta configuración utiliza un proveedor de estado solo en memoria y establece el número de réplicas en 1.

### <a name="defaults-and-generated-settings"></a>Valores predeterminados y configuración generada
Cuando se usa el atributo `StatePersistence`, se selecciona automáticamente un proveedor de estado en tiempo de ejecución cuando se inicia el servicio de actor. Sin embargo, el número de réplicas, se establece en tiempo de compilación con las herramientas de compilación de actores de Visual Studio. Las herramientas de compilación generan automáticamente un *servicio predeterminado* del servicio de actor en ApplicationManifest.xml. Los parámetros se crean para **min replica set size** y **target replica set size**.

Puede cambiar estos parámetros manualmente. Sin embargo, cada vez que se cambia el atributo `StatePersistence`, los parámetros se establecen, de forma predeterminada, en los valores del tamaño del conjunto de réplicas para el atributo `StatePersistence`, invalidando los valores anteriores. Es decir, los valores establecidos en ServiceManifest.xml *solo* se reemplazan al compilar cuando cambia el valor del atributo `StatePersistence`.

```xml
<ApplicationManifest xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="Application12Type" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
   <Parameters>
      <Parameter Name="MyActorService_PartitionCount" DefaultValue="10" />
      <Parameter Name="MyActorService_MinReplicaSetSize" DefaultValue="3" />
      <Parameter Name="MyActorService_TargetReplicaSetSize" DefaultValue="3" />
   </Parameters>
   <ServiceManifestImport>
      <ServiceManifestRef ServiceManifestName="MyActorPkg" ServiceManifestVersion="1.0.0" />
   </ServiceManifestImport>
   <DefaultServices>
      <Service Name="MyActorService" GeneratedIdRef="77d965dc-85fb-488c-bd06-c6c1fe29d593|Persisted">
         <StatefulService ServiceTypeName="MyActorServiceType" TargetReplicaSetSize="[MyActorService_TargetReplicaSetSize]" MinReplicaSetSize="[MyActorService_MinReplicaSetSize]">
            <UniformInt64Partition PartitionCount="[MyActorService_PartitionCount]" LowKey="-9223372036854775808" HighKey="9223372036854775807" />
         </StatefulService>
      </Service>
   </DefaultServices>
</ApplicationManifest>
```

## <a name="state-manager"></a>Administrador de estado
Cada instancia de actor tiene su propio administrador de estado: una estructura de datos similar a un diccionario que almacena de manera confiable los pares clave-valor. El administrador de estado es un contenedor alrededor de un proveedor de estado. Se puede utilizar para almacenar datos, con independencia de la configuración de persistencia que se utilice, pero no proporciona ninguna garantía de que se pueda cambiar un servicio de actor en ejecución de una configuración de estado volátil (solo en memoria) a una configuración de estado persistente mediante una actualización gradual mientras se conservan los datos. Sin embargo, es posible cambiar el número de réplicas para un servicio en ejecución.

Las claves del administrador de estado deben ser cadenas. Los valores son genéricos y pueden ser de cualquier tipo, incluidos los tipos personalizados. Los valores almacenados en el administrador de estado deben ser serializables en el contrato de datos, ya que pueden transmitirse a través de la red a otros nodos durante la replicación y pueden escribirse en el disco, dependiendo de la configuración de persistencia del estado de un actor.

El administrador de estado expone los métodos de diccionario comunes para administrar el estado, de manera similar a los que se encuentran en un diccionario confiable.

Para obtener ejemplos de administración del estado de actor, lea [Access, save, and remove Reliable Actors state](service-fabric-reliable-actors-access-save-remove-state.md) (Acceder al estado de Reliable Actors, guardarlo y quitarlo).

## <a name="best-practices"></a>Procedimientos recomendados
A continuación se incluyen diversas prácticas recomendadas y sugerencias de resolución de problemas para administrar el estado de los actores.

### <a name="make-the-actor-state-as-granular-as-possible"></a>Hacer que el estado de los actores sea lo más pormenorizado posible
Esto es fundamental para el rendimiento y el uso de recursos de la aplicación. Siempre que se escriba o se actualice el "estado con nombre" de un actor, todo el valor correspondiente a ese "estado con nombre" se serializa y se envía a través de la red a las réplicas secundarias.  Los servidores secundarios lo escriben en el disco local y responden a la réplica principal. Cuando la principal recibe confirmaciones de un cuórum de réplicas secundarias, escribe el estado en su disco local. Por ejemplo, suponga que el valor es una clase que tiene 20 miembros y un tamaño de 1 MB. Incluso si solo modifica uno de los miembros de la clase con un tamaño de 1 KB, acabará por pagar el costo de la serialización y las escrituras de disco y red para 1 MB completo. Del mismo modo, si el valor es una colección (como una lista, matriz o diccionario), paga el costo por toda la colección incluso si solo modifica uno de sus miembros. La interfaz de StateManager de la clase de actor es como un diccionario. Siempre se debe modelar la estructura de datos que representa el estado de actor sobre este diccionario.
 
### <a name="correctly-manage-the-actors-life-cycle"></a>Administrar correctamente el ciclo de vida del actor
Debe tener una directiva clara acerca de cómo administrar el tamaño del estado en cada partición de un servicio de actor. El servicio de actor debe tener un número fijo de actores y reutilizarlos tanto como sea posible. Si crea continuamente nuevos actores, debe eliminarlos una vez que hayan terminado su trabajo. La plataforma de actores almacena algunos metadatos sobre cada actor que existe. La eliminación de todo el estado de un actor no elimina los metadatos sobre ese actor. Debe eliminar el actor (consulte cómo [eliminar actores y su estado](service-fabric-reliable-actors-lifecycle.md#manually-deleting-actors-and-their-state)) para eliminar toda la información sobre dicho actor almacenada en el sistema. Como comprobación adicional, debe consultar el servicio de actor (consulte [Enumeración de los actores](service-fabric-reliable-actors-enumerate.md)) cada cierto tiempo para asegurarse de que el número de actores está dentro del intervalo esperado.
 
Si alguna vez observa que el tamaño del archivo de base de datos de un servicio de actor supera el tamaño previsto, compruebe que se respeten las directrices anteriores. Si sigue estas directrices y aún tiene problemas con el tamaño del archivo de base de datos, debe [abrir una incidencia de soporte técnico](service-fabric-support.md) con el equipo de producto para obtener ayuda.

## <a name="next-steps"></a>Pasos siguientes

Un estado que se almacena en Reliable Actors debe serializarse antes de escribirse en el disco y replicarse para lograr alta disponibilidad. Aprenda más sobre la [serialización del tipo de actor](service-fabric-reliable-actors-notes-on-actor-type-serialization.md).

A continuación, aprenda más en [Supervisión del rendimiento y diagnósticos de los actores](service-fabric-reliable-actors-diagnostics.md).
