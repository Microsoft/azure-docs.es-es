---
title: Introducción a Reliable Actors de Service Fabric
description: Introducción al modelo de programación de Reliable Actors de Service Fabric, basado en el patrón Virtual Actor.
author: vturecek
ms.topic: conceptual
ms.date: 11/01/2017
ms.author: vturecek
ms.custom: devx-track-csharp
ms.openlocfilehash: adb15d995cd2a9fd604aa6b91360adc88a2804e6
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89007934"
---
# <a name="introduction-to-service-fabric-reliable-actors"></a>Introducción a Service Fabric Reliable Actors.
Reliable Actors es un marco de trabajo de la aplicación de Service Fabric que se basa en el modelo [Virtual Actor](https://research.microsoft.com/en-us/projects/orleans/) . La API de Reliable Actors proporciona un modelo de programación uniproceso basado en las garantías de escalabilidad y confiabilidad que ofrece Service Fabric.

## <a name="what-are-actors"></a>¿Qué son los actores?
Un actor es una unidad aislada e independiente de proceso y estado con ejecución uniproceso. El [patrón de actor](https://en.wikipedia.org/wiki/Actor_model) es un modelo informático para sistemas simultáneos o distribuidos en que un gran número de estos actores puede ejecutarse simultánea e independientemente unos de otros. Los actores pueden comunicarse entre sí y pueden crear más actores.

### <a name="when-to-use-reliable-actors"></a>Cuándo usar Reliable Actors
Service Fabric Reliable Actors es una implementación del modelo de diseño del actor. Al igual que sucede con cualquier modelo de diseño de software, la decisión sobre si usar un modelo específico se toma en función de si un problema de diseño de software se adapta al modelo o no.

Aunque el modelo de diseño del actor puede resultar adecuado para un número de problemas y escenarios de sistemas distribuidos, se requiere una consideración cuidadosa de las restricciones del modelo y el marco de trabajo que lo implementa. Como regla general, tenga en cuenta el patrón de actor para modelar su problema o escenario si:

* El espacio del problema implica un gran número (miles o más) de unidades pequeñas, independientes y aisladas de estado y lógica.
* Desea trabajar con objetos uniproceso que no requieren una interacción importante de los componentes externos, incluida la consulta del estado en un conjunto de actores.
* Las instancias de actor no bloquearán a los autores de llamadas con retrasos imprevisibles emitiendo operaciones de E/S.

## <a name="actors-in-service-fabric"></a>Actores en Service Fabric
En Service Fabric, los actores se implementan en el marco de Reliable Actors: Un marco de trabajo de la aplicación basado en patrones construido sobre [Reliable Services de Service Fabric](service-fabric-reliable-services-introduction.md). Cada servicio de Reliable Actors que escribe es en realidad un servicio de confianza con estado particionado.

Cada actor se define como una instancia de un tipo de actor, similar a la forma en que un objeto de .NET es una instancia de un tipo de .NET. Por ejemplo, puede haber un tipo de actor que implemente la funcionalidad de una calculadora y podría haber muchos actores de ese tipo que se distribuyesen en varios nodos en un clúster. Cada uno de esos actores se identifica de forma única mediante un identificador de actor.

## <a name="actor-lifetime"></a>Duración del actor
Los actores de Service Fabric son virtuales, lo que significa que su duración no está vinculada a su representación en memoria. Como resultado, no es necesario crearlas ni destruirlas explícitamente. El tiempo de ejecución de Reliable Actors activa automáticamente un actor la primera vez que recibe una solicitud para ese identificador de actor. Si no se usa un actor durante un período de tiempo, el tiempo de ejecución de Reliable Actors recopila el objeto en memoria como elemento no utilizado. También mantendrá conocimientos de la existencia del actor por si es necesario reactivarlo más adelante. Para más información, consulte [Ciclo de vida de un actor y recolección de elementos no utilizados](service-fabric-reliable-actors-lifecycle.md).

Esta abstracción de la duración del actor virtual incluye algunas advertencias como resultado del modelo Virtual Actor y, de hecho, la implementación de Reliable Actors se desvía en ocasiones de este modelo.

* Un actor se activa automáticamente (provocando la creación de un objeto de actor) la primera vez que se envía un mensaje a su identificador de actor. Transcurrido un período de tiempo, el objeto de actor se recolecta como elemento no utilizado. En el futuro, usar de nuevo el identificador de actor provoca la creación de un nuevo objeto de actor. La duración del estado del actor equivale a la vida útil del objeto al almacenarse en el administrador de estado.
* Al llamar a cualquier método de actor para un identificador de actor se activa dicho actor. Por este motivo, se llama de forma implícita al constructor de los tipos de actor mediante el tiempo de ejecución. Por lo tanto, el código de cliente no puede pasar parámetros al constructor del tipo de actor, aunque se pueden pasar parámetros al constructor del actor mediante el propio servicio. El resultado es que los actores se pueden crear en un estado inicializado de forma parcial en el momento en que se llama a otros métodos en él, en caso de que el actor requiera parámetros de inicialización del cliente. No hay ningún punto de entrada único para la activación de un actor desde el cliente.
* Aunque Reliable Actors crea de forma implícita objetos de actor, tiene la posibilidad de eliminar explícitamente un actor y su estado.

## <a name="distribution-and-failover"></a>Distribución y conmutación por error
Para proporcionar escalabilidad y confiabilidad, Service Fabric distribuye actores en el clúster y los migra automáticamente desde los nodos con errores hasta los correctos según sea necesario. Se trata de una abstracción sobre un [servicio de confianza con estado particionado](service-fabric-concepts-partitioning.md). Distribución, escalabilidad, confiabilidad y conmutación por error automática se proporcionan en virtud del hecho de que los actores se ejecuten en un servicio de confianza con estado denominado *Servicio de actor*.

Los actores se distribuyen entre las particiones del Servicio de actor y dichas particiones se distribuyen entre los nodos en un clúster de Service Fabric. Cada partición del servicio contiene un conjunto de actores. Service Fabric administra la distribución y conmutación por error de las particiones del servicio.

Por ejemplo, un Servicio de actor con nueve particiones implementadas en tres nodos mediante la ubicación predeterminada de la partición del actor se distribuiría del siguiente modo:

![Distribución de Reliable Actors][2]

El marco de trabajo de actores administra la configuración del intervalo de claves y el esquema de partición por usted. Esto simplifica algunas opciones, pero también implica cierta consideración:

* Reliable Services le permite elegir un esquema de creación de particiones, el intervalo de claves (al usar un esquema de creación de particiones por rangos) y el recuento de particiones. Reliable Actors se restringe al esquema de creación de particiones por rangos (el esquema Int64 uniforme) y requiere que use el intervalo de claves Int64 completo.
* De forma predeterminada, los actores se colocan aleatoriamente en particiones, dando como resultado una distribución uniforme.
* Como los actores se colocan aleatoriamente, debe esperarse que las operaciones del actor requieran siempre la comunicación de red, incluidas la serialización y deserialización de datos de la llamada al método, incurriendo en latencia y sobrecarga.
* En escenarios avanzados, es posible controlar la ubicación de la partición del actor mediante los identificadores de actor Int64 que se asignan a particiones específicas. Sin embargo, esto puede dar lugar a una distribución desequilibrada de los actores entre particiones.

Para más información sobre cómo se particionan los servicios de actor, consulte [Partitioning concepts for actors](service-fabric-reliable-actors-platform.md#service-fabric-partition-concepts-for-actors)(Conceptos de creación de particiones para actores).

## <a name="actor-communication"></a>Comunicación con actores
Las interacciones entre actores se definen en una interfaz compartida por el actor que implementa la interfaz y el cliente que consigue un proxy a un actor a través de la misma interfaz. Como esta interfaz se usa para invocar métodos de actor de forma asincrónica, cada método de la interfaz debe devolver tareas.

Las invocaciones de método y sus respuestas tienen como resultado final solicitudes de red en el clúster, de modo que los argumentos y los tipos de resultado de las tareas que devuelven deben ser serializables por parte de la plataforma. En particular, deben ser [serializables mediante contrato de datos](service-fabric-reliable-actors-notes-on-actor-type-serialization.md).

### <a name="the-actor-proxy"></a>El proxy de actor
La API de cliente de Reliable Actors ofrece comunicación entre una instancia de actor y un cliente de actor. Para comunicarse con un actor, un cliente crea un objeto proxy de actor que implementa la interfaz del actor. El cliente interactúa con el actor mediante la invocación de métodos en el objeto proxy. El proxy de actor puede usarse para la comunicación de cliente a actor y de actor a actor.

```csharp
// Create a randomly distributed actor ID
ActorId actorId = ActorId.CreateRandom();

// This only creates a proxy object, it does not activate an actor or invoke any methods yet.
IMyActor myActor = ActorProxy.Create<IMyActor>(actorId, new Uri("fabric:/MyApp/MyActorService"));

// This will invoke a method on the actor. If an actor with the given ID does not exist, it will be activated by this method call.
await myActor.DoWorkAsync();
```

```java
// Create actor ID with some name
ActorId actorId = new ActorId("Actor1");

// This only creates a proxy object, it does not activate an actor or invoke any methods yet.
MyActor myActor = ActorProxyBase.create(actorId, new URI("fabric:/MyApp/MyActorService"), MyActor.class);

// This will invoke a method on the actor. If an actor with the given ID does not exist, it will be activated by this method call.
myActor.DoWorkAsync().get();
```


Tenga en cuenta que los dos tipos de datos que se usan para crear el objeto de proxy de actor son el identificador del actor y el nombre de la aplicación. El identificador de actor identifica de forma única al actor, mientras que el nombre de la aplicación identifica la [Aplicación de Service Fabric](service-fabric-reliable-actors-platform.md#application-model) en la que se implementa el actor.

La clase `ActorProxy`(C#) / `ActorProxyBase`(Java) del cliente realiza la resolución necesaria para localizar el actor mediante el identificador y abre un canal de comunicación con él. También vuelve a intentar localizar el actor en caso de que se produzcan errores de comunicación y conmutaciones por error. Como resultado, la entrega de mensajes tiene las siguientes características:

* La entrega de mensajes es la mejor opción.
* Los actores pueden recibir mensajes duplicados del mismo cliente.

## <a name="concurrency"></a>Simultaneidad
El tiempo de ejecución de Reliable Actors ofrece un modelo de acceso simple basado en turnos para obtener acceso a los métodos de actor. Esto significa que no puede haber más de un subproceso activo dentro del código del objeto de actor en ningún momento. El acceso basado en turnos simplifica considerablemente sistemas simultáneos, pues no es necesario que los mecanismos de sincronización tengan acceso a datos. También significa que los sistemas deben diseñarse con consideraciones especiales para la naturaleza de acceso uniproceso de cada instancia de actor.

* Una sola instancia de actor no puede procesar más de una solicitud cada vez. Una instancia de actor puede provocar un cuello de botella de procesamiento si se espera que controle solicitudes simultáneas.
* Los actores pueden interbloquearse si hay una solicitud circular entre dos actores mientras se realiza una solicitud externa destinada a uno de los actores simultáneamente. El tiempo de ejecución del actor agotará el tiempo de expiración en llamadas de actor automáticamente y generará una excepción al autor de llamada para interrumpir posibles situaciones de interbloqueo.

![Comunicación de Reliable Actors][3]

### <a name="turn-based-access"></a>Acceso basada en turnos
Un turno es la ejecución completa de un método de actor en respuesta a la solicitud de otros actores o clientes, o la ejecución completa de la devolución de llamada de un [temporizador o recordatorio](service-fabric-reliable-actors-timers-reminders.md) . Aunque estos métodos y devoluciones de llamada son asincrónicos, el tiempo de ejecución de los actores no los intercala. Un turno debe completarse por completo antes de que se permita uno nuevo. En otras palabras, la devolución de llamada de un método de actor, un temporizador o recordatorio que se esté ejecutando se debe completar totalmente antes de que se permita una llamada a un método o una devolución de llamada nuevas. Un método o una devolución de llamada se consideran finalizadas si la ejecución ha devuelto desde el método o la devolución de llamada y la tarea devuelta por el método o la devolución de llamada ha terminado. Merece la pena resaltar que la simultaneidad basada en turnos se respeta incluso entre devoluciones de llamada, temporizadores y métodos diferentes.

El tiempo de ejecución de los actores impone la simultaneidad basada en turnos mediante la adquisición de un bloqueo por actor al principio de un turno y su liberación al final del turno. Por lo tanto, la simultaneidad basada en turnos se aplica por actor y no para todos los actores. Las devoluciones de llamada de los métodos de actor, los temporizadores y los recordatorios se pueden ejecutar simultáneamente en nombre de los diferentes actores.

En el ejemplo siguiente se muestran los conceptos anteriores. Supongamos un tipo de actor que implementa dos métodos asincrónicos (digamos *Method1* y *Method2*), un temporizador y un recordatorio. En el diagrama siguiente se muestra un ejemplo de una escala de tiempo de la ejecución de estos métodos y las devoluciones de llamada en nombre de los dos actores (*ActorId1* y *ActorId2*) que pertenecen a este tipo de actor.

![Acceso y simultaneidad basada en turnos de tiempo de ejecución de Reliable Actors][1]

Este diagrama sigue estas convenciones:

* Cada línea vertical muestra el flujo lógico de ejecución de un método o una devolución de llamada en nombre de un actor determinado.
* Los eventos marcados en cada línea vertical se producen en orden cronológico, con los eventos más recientes por debajo de los más antiguos.
* Se usan colores diferentes para las escalas de tiempo correspondientes a actores distintos.
* Se usa el resaltado para indicar la duración durante la cual se mantiene el bloqueo por actor en nombre de un método o una devolución de llamada.

Algunos puntos importantes a tener en cuenta:

* Cuando *Method1* se ejecuta en nombre de *ActorId2* en respuesta a la solicitud de cliente *xyz789*, llega otra solicitud de cliente (*abc123*) que también requiere que *Method1* lo ejecute *ActorId2*. Sin embargo, la segunda ejecución de *Method1* no comienza hasta que se haya completado la ejecución anterior. De forma similar, un recordatorio que haya registrado *ActorId2* se activa cuando *Method1* se ejecuta en respuesta a la solicitud de cliente *xyz789*. La devolución de llamada del recordatorio se ejecuta solo después de que las dos ejecuciones de *Method1* se hayan completado. Todo esto se debe a la simultaneidad basada en turnos que se exige para *ActorId2*.
* De forma similar, también se aplica la simultaneidad basada en turnos para *ActorId1*, como se muestra en la ejecución de *Method1*, la ejecución de *Method2* y la devolución de llamada del temporizador en nombre de *ActorId1*, que suceden en serie.
* La ejecución de *Method1* en nombre de *ActorId1* se superpone con su ejecución en nombre de *ActorId2*. Esto es así porque solo se exige la simultaneidad basada en turnos dentro de un actor y no para todos los actores.
* En algunas ejecuciones de métodos y devoluciones de llamada, el elemento `Task`(C#) / `CompletableFuture`(Java) que devuelve el método o la devolución de llamada se completa después de la devolución del método. En otras, la operación asincrónica ya se completó antes de la devolución del método o la devolución de llamada. En ambos casos, el bloqueo por actor se libera solo después de la devolución del método o la devolución de llamada, y de la finalización de la operación asincrónica.

### <a name="reentrancy"></a>Reentrada
El tiempo de ejecución de los actores permite la reentrada de forma predeterminada. Esto significa que si un método de actor del *actor A* llama a un método del *actor B*, que a su vez llama a otro método del *actor A*, se permite que ese método se ejecute. Esto es porque forma parte del mismo contexto lógico de la cadena de llamada. Todas las llamadas del temporizador y el recordatorio comienzan con el nuevo contexto de llamada lógico. Consulte [Reentrada en Reliable Actors](service-fabric-reliable-actors-reentrancy.md) para más detalles.

### <a name="scope-of-concurrency-guarantees"></a>Ámbito de garantías de simultaneidad
El tiempo de ejecución de los actores ofrece estas garantías de simultaneidad en situaciones donde controla la invocación de estos métodos. Por ejemplo, ofrece estas garantías para las invocaciones de método que se realizan en respuesta a una solicitud de cliente y para las devoluciones de llamada de temporizadores y recordatorios. Sin embargo, si el código del actor invoca directamente a estos métodos fuera de los mecanismos que ofrece el tiempo de ejecución de los actores, el tiempo de ejecución no puede ofrecer ninguna garantía de simultaneidad. Por ejemplo, si el método se invoca en el contexto de una tarea que no está asociada con la tarea que han devuelto los métodos de actor, el tiempo de ejecución no puede ofrecer garantías de simultaneidad. Si el método se invoca desde un subproceso que el actor crea por sí mismo, entonces el tiempo de ejecución tampoco puede proporcionar garantías de simultaneidad. Por lo tanto, para realizar operaciones en segundo plano, los actores deben usar [temporizadores de actor o recordatorios de actor](service-fabric-reliable-actors-timers-reminders.md) que respeten la simultaneidad basada en turnos.

## <a name="next-steps"></a>Pasos siguientes
Para empezar, cree su primer servicio de Reliable Actors:
   * [Introducción a Reliable Actors en .NET](service-fabric-reliable-actors-get-started.md)
   * [Introducción a Reliable Actors en Java](./service-fabric-create-your-first-linux-application-with-java.md)

<!--Image references-->
[1]: ./media/service-fabric-reliable-actors-introduction/concurrency.png
[2]: ./media/service-fabric-reliable-actors-introduction/distribution.png
[3]: ./media/service-fabric-reliable-actors-introduction/actor-communication.png
