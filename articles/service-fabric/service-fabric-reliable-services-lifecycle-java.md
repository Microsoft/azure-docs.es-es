---
title: Ciclo de vida de Reliable Services de Azure Service Fabric
description: Obtenga información sobre los eventos de ciclo de vida de una aplicación de Azure Service Fabric Reliable Services con Java para servicios con estado y sin estado.
author: PavanKunapareddyMSFT
ms.topic: conceptual
ms.date: 06/30/2017
ms.custom: devx-track-java
ms.author: pakunapa
ms.openlocfilehash: b22c78a0259e4430ac6bfae1c0a9379c4a832cd4
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87324614"
---
# <a name="reliable-services-lifecycle"></a>Ciclo de vida de Reliable Services
> [!div class="op_single_selector"]
> * [C# en Windows](service-fabric-reliable-services-lifecycle.md)
> * [Java en Linux](service-fabric-reliable-services-lifecycle-java.md)
>
>

Reliable Services es uno de los modelos de programación disponibles en Azure Service Fabric. Cuando se aprende sobre el ciclo de vida de Reliable Services, es muy importante comprender los eventos básicos del ciclo de vida. El orden exacto de los eventos depende de los detalles de configuración. 

En general, el ciclo de vida de Reliable Services incluye los siguientes eventos:

* Durante el inicio:
  * Se construyen los servicios.
  * Los servicios tienen la oportunidad de construir y devolver cero o más agentes de escucha.
  * Se abre cualquier agente de escucha devuelto para la comunicación con el servicio.
  * Se llama al método `runAsync` del servicio para que el servicio pueda realizar trabajos de ejecución prolongada o en segundo plano.
* Durante el cierre:
  * Se cancela el token de cancelación pasado a `runAsync` y se cierran los agentes de escucha.
  * Se destruye el propio objeto del servicio.

El orden de los eventos en Reliable Services puede cambiar ligeramente en función de si la instancia es sin estado o con estado. 

Además, en los servicios con estado, debe resolver el escenario de intercambio principal. Durante esta secuencia, el rol principal se transfiere a otra réplica (o se devuelve) sin que el servicio tenga que cerrarse. 

Por último, hay que pensar en las condiciones de error.

## <a name="stateless-service-startup"></a>Inicio de un servicio sin estado
El ciclo de vida de un servicio sin estado es bastante sencillo. Este es el orden de los eventos:

1. Se construye el servicio.
2. Estos eventos se producen en paralelo:
    - Se invoca `StatelessService.createServiceInstanceListeners()` y se abren los agentes de escucha devueltos. Se llama a `CommunicationListener.openAsync()` en cada uno de los agentes de escucha.
    - Se llama al método `runAsync` del servicio (`StatelessService.runAsync()`).
3. Si está presente, se llama al propio método `onOpenAsync` del servicio. En concreto, se llama a `StatelessService.onOpenAsync()`. Esta invalidación es poco habitual, pero está disponible.

Es importante tener en cuenta que no hay ninguna ordenación entre las llamadas para crear y abrir los agentes de escucha y la llamada a `runAsync`. Los agentes de escucha podrían abrirse antes de que se inicie `runAsync`. De forma similar, `runAsync` podría ser invocado antes de que se abran o incluso de que se hayan construido los agentes de escucha de comunicación. El implementador deberá llevar a cabo cualquier sincronización necesaria. Estas son algunas de las soluciones habituales:

* En ocasiones, los agentes de escucha no pueden funcionar hasta que se crea otra información o se realiza otro trabajo. En los servicios sin estado ese trabajo se suele realizar en el constructor del servicio. Se puede realizar durante la llamada a `createServiceInstanceListeners()` o como parte de la construcción del propio agente de escucha.
* En ocasiones, el código de `runAsync` no se inicia hasta que están abiertos los agentes de escucha. En este caso, hace falta coordinación adicional. Una solución habitual consiste en agregar una marca en los agentes de escucha. La marca indica cuándo han terminado los agentes de escucha. El método `runAsync` comprueba esto antes de continuar el trabajo real.

## <a name="stateless-service-shutdown"></a>Cierre de un servicio sin estado
Al cerrar un servicio sin estado, se sigue el mismo patrón, solo que en orden inverso:

1. Estos eventos se producen en paralelo:
    - Todos los agentes de escucha abiertos se cierran. Se llama a `CommunicationListener.closeAsync()` en cada uno de los agentes de escucha.
    - Se cancela el token de cancelación que se pasó a `runAsync()`. La comprobación de la propiedad `isCancelled` del token de cancelación devuelve `true` y, si se llama al método `throwIfCancellationRequested` del token, se inicia una excepción `CancellationException`.
2. Una vez que `closeAsync()` finaliza en cada agente de escucha y tras finalizar también `runAsync()`, se llama al método `StatelessService.onCloseAsync()` del servicio, si está presente. Nuevamente, esta no es una invalidación común, pero puede utilizarse para cerrar de manera segura los recursos, detener el procesamiento en segundo plano, terminar de guardar el estado externo o cerrar las conexiones existentes.
3. Una vez finalizado `StatelessService.onCloseAsync()`, se destruye el objeto de servicio.

## <a name="stateful-service-startup"></a>Inicio de un servicio con estado
Los servicios con estado tienen un patrón similar al de los servicios sin estado, con unos pocos cambios.  Al iniciarse un servicio con estado, el orden de los eventos es el siguiente:

1. Se construye el servicio.
2. Se llama a `StatefulServiceBase.onOpenAsync()`. Por lo general, la llamada no se invalida en el servicio.
3. Estos eventos se producen en paralelo:
    - Se invoca a `StatefulServiceBase.createServiceReplicaListeners()`. 
      - Si se trata de un servicio principal, se abren todos los agentes de escucha devueltos. Se llama a `CommunicationListener.openAsync()` en cada uno de los agentes de escucha.
      - Si se trata de un servicio secundario, solamente se abren los agentes de escucha con la marca `listenOnSecondary = true`. No es muy frecuente tener agentes de escucha abiertos en servicios secundarios.
    - Si en la actualidad el servicio es un servicio principal, se llama al método `StatefulServiceBase.runAsync()` del servicio.
4. Una vez que se han completado todas las llamadas a `openAsync()` de los agentes de escucha de la réplica y se ha llamando a `runAsync()`, se llama a `StatefulServiceBase.onChangeRoleAsync()`. Por lo general, la llamada no se invalida en el servicio.

Al igual que en los servicios sin estado, en los servicios con estado no existe relación entre el orden en el que se crean y se abren los agentes de escucha y el momento en que se llama a `runAsync`. Si necesita que estas operaciones estén coordinadas, las soluciones serían prácticamente iguales. Hay un caso adicional en los servicios con estado. Imaginemos que las llamadas que llegan a los agentes de escucha de comunicación requieren que la información se mantenga dentro de alguna instancia de [Reliable Collections](service-fabric-reliable-services-reliable-collections.md). Dado que es posible que los agentes de escucha de comunicación se abrieran antes de que se pudiera leer o escribir en las colecciones confiables y antes de que se inicie `runAsync`, se necesita coordinación adicional. La solución más sencilla y habitual consiste en que los agentes de escucha de comunicación devuelvan un código de error. El cliente utiliza el código de error para saber si volver a intentar la solicitud.

## <a name="stateful-service-shutdown"></a>Cierre de un servicio con estado
De forma similar a los servicios sin estado, los eventos de ciclo de vida durante el cierre son los mismos que durante el inicio, pero a la inversa. Cuando un servicio con estado se está cerrando, se producen los siguientes eventos:

1. Estos eventos se producen en paralelo:
    - Todos los agentes de escucha abiertos se cierran. Se llama a `CommunicationListener.closeAsync()` en cada uno de los agentes de escucha.
    - Se cancela el token de cancelación que se pasó a `runAsync()`. Una llamada al método `isCancelled()` del token de cancelación devuelve `true` y, si se llama al método `throwIfCancellationRequested()` del token, se inicia una excepción `OperationCanceledException`.
2. Una vez que `closeAsync()` finaliza en cada agente de escucha y tras finalizar también `runAsync()`, se llama al método `StatefulServiceBase.onChangeRoleAsync()` del servicio. Por lo general, la llamada no se invalida en el servicio.

   > [!NOTE]  
   > Solo es necesario esperar a que `runAsync` finalice si esta réplica es una réplica principal.

3. Una vez que finaliza el método `StatefulServiceBase.onChangeRoleAsync()`, se llama al método `StatefulServiceBase.onCloseAsync()`. Esta llamada es una invalidación poco habitual, pero está disponible.
3. Una vez finalizado `StatefulServiceBase.onCloseAsync()`, se destruye el objeto de servicio.

## <a name="stateful-service-primary-swaps"></a>Intercambios de servicios con estado principales
Durante la ejecución de un servicio con estado, solo se abren los agentes de escucha de comunicación y se llama al método `runAsync` en las réplicas principales de esos servicios con estado. Se construyen réplicas secundarias, pero no reciben más llamadas. Durante la ejecución de un servicio con estado la réplica que es actualmente la principal puede cambiar. Los eventos del ciclo de vida que un réplica con estado puede ver dependen de si es la réplica que está subiendo o bajando de categoría.

### <a name="for-the-demoted-primary"></a>Para la réplica principal degradada
Service Fabric necesita que la réplica principal degradada deje de procesar mensajes y que cierre cualquier trabajo que esté realizando en segundo plano. Este paso es similar a cuando se cierra el servicio. Se diferencia en que el servicio no se destruye ni se cierra, puesto que permanece como secundario. Se producen los siguientes eventos:

1. Estos eventos se producen en paralelo:
    - Todos los agentes de escucha abiertos se cierran. Se llama a `CommunicationListener.closeAsync()` en cada uno de los agentes de escucha.
    - Se cancela el token de cancelación que se pasó a `runAsync()`. Una comprobación del método `isCancelled()` del token de cancelación devuelve `true`. Si se le llama, el método `throwIfCancellationRequested()` del token produce una excepción `OperationCanceledException`.
2. Una vez que `closeAsync()` finaliza en cada agente de escucha y tras finalizar también `runAsync()`, se llama al método `StatefulServiceBase.onChangeRoleAsync()` del servicio. Por lo general, la llamada no se invalida en el servicio.

### <a name="for-the-promoted-secondary"></a>Para la réplica secundaria promovida
De igual modo, Service Fabric necesita que la réplica secundaria que sube de categoría comience a escuchar los mensajes de la conexión e inicie las tareas en segundo plano que debe completar. Este proceso es similar a cuando se crea el servicio. La diferencia es que la propia réplica ya existe. Se producen los siguientes eventos:

1. Estos eventos se producen en paralelo:
    - Se invoca `StatefulServiceBase.createServiceReplicaListeners()` y se abren los agentes de escucha devueltos. Se llama a `CommunicationListener.openAsync()` en cada uno de los agentes de escucha.
    - Se llama al método `StatefulServiceBase.runAsync()` del servicio.
2. Una vez que se han completado todas las llamadas a `openAsync()` de los agentes de escucha de la réplica y se ha llamando a `runAsync()`, se llama a `StatefulServiceBase.onChangeRoleAsync()`. Por lo general, la llamada no se invalida en el servicio.

### <a name="common-issues-during-stateful-service-shutdown-and-primary-demotion"></a>Problemas habituales durante la degradación de la categoría principal y el cierre de un servicio con estado
Service Fabric cambia la categoría principal de un servicio con estado por diferentes razones. Las razones más frecuente son el [reequilibrio de clústeres](service-fabric-cluster-resource-manager-balancing.md) y la [actualización de aplicaciones](service-fabric-application-upgrade.md). Durante estas operaciones, es importante que el servicio respete el `cancellationToken`. Esto también se aplica durante el cierre normal del servicio, por ejemplo, si se ha eliminado el servicio.

Los servicios que no administren correctamente la cancelación pueden experimentar errores. El rendimiento de estas operaciones es lento, ya que Service Fabric espera que los servicios se detengan sin que se produzcan errores. En última instancia, esto puede producir errores en las actualizaciones, que pueden agotar el tiempo de espera y revertirse. Si no se respeta el token de cancelación, también puede producirse un desequilibrio en los clústeres. Los clústeres pueden llegar a estar desequilibrados porque exceso de carga en los nodos. Sin embargo, los servicios no se vuelven a equilibrar porque es demasiado lento moverlos a otra parte. 

Como los servicios tienen estado, también es probable que utilicen [Colecciones confiables](service-fabric-reliable-services-reliable-collections.md). En Service Fabric, cuando un servicio principal se degrada de nivel, una de las primeras cosas que ocurren es que se revoca el acceso de escritura al estado subyacente. Esto genera un segundo conjunto de problemas que pueden afectar al ciclo de vida del servicio. Las colecciones devuelven excepciones en función del momento elegido y de si la réplica se está transfiriendo a otro lugar o cerrando. Es importante controlar estas excepciones correctamente. 

Las excepciones producidas por Service Fabric son permanentes [(`FabricException`)](/java/api/system.fabric.exception) o transitorias [(`FabricTransientException`)](/java/api/system.fabric.exception.fabrictransientexception). Las excepciones permanentes deberían ser registras y lanzadas. Las excepciones transitorias se pueden volver a intentar en función de una lógica de reintento.

Administrar las excepciones que proceden del uso de `ReliableCollections` junto con los eventos del ciclo de vida del servicio es una parte importante del proceso de comprobación y validación de un servicio de Reliable Services. Se recomienda que ejecute siempre el servicio bajo carga. También debe realizar las actualizaciones y las [pruebas de caos](service-fabric-controlled-chaos.md) antes de la implementación en producción. Estos pasos básicos ayudan a garantizar que el servicio está implementado correctamente y que puede administrar los eventos del ciclo de vida correctamente.

## <a name="notes-on-service-lifecycle"></a>Notas sobre el ciclo de vida del servicio
* Tanto el método `runAsync()` como las llamadas a `createServiceInstanceListeners/createServiceReplicaListeners` son opcionales. Un servicio puede tener uno, ambos o ninguno. Por ejemplo, si el servicio realiza todo su trabajo en respuesta a las llamadas del usuario, no es necesario que implemente `runAsync()`. Solo son necesarios los agentes de escucha de comunicación y el código asociado.  Del mismo modo, crear y devolver agentes de escucha de comunicación es opcional. El servicio puede tener solo trabajos en segundo plano, por lo que solo es necesario implementar `runAsync()`.
* Un servicio puede completar `runAsync()` correctamente y volver desde este. Esto no se considera una condición de error. Representa la finalización del trabajo en segundo plano del servicio. Para los servicios de Reliable Services con estado, se llamaría a `runAsync()` de nuevo si el servicio pasara de principal a secundario y de nuevo a principal.
* Si un servicio sale de `runAsync()` iniciando una excepción inesperada, se considera un error. El objeto del servicio se cierra y se notifica un error de mantenimiento.
* Aunque no hay un límite de tiempo para que se devuelvan resultados de estos métodos, se pierde inmediatamente la posibilidad de escribir. Por lo tanto, no se puede completar ningún trabajo real. Se recomienda que devuelva resultados lo antes posible tras recibir la solicitud de cancelación. Si su servicio no responde a estas llamadas a la API en un intervalo de tiempo razonable, Service Fabric puede forzar la finalización del servicio. Esto solo suele pasar durante las actualizaciones de aplicaciones o cuando se elimina un servicio. Este tiempo de espera es de 15 minutos de manera predeterminada.
* Los errores en la ruta de acceso de `onCloseAsync()` provocan la llamada a `onAbort()`. Esta llamada es una oportunidad como último recurso para hacer todo lo posible para que el servicio limpie y libere cualquier recurso que haya reclamado. Normalmente se llama cuando se detecta un error permanente en el nodo o cuando Service Fabric no puede administrar el ciclo de vida de la instancia de servicio debido a errores internos.
* Se llama a `OnChangeRoleAsync()` cuando la réplica del servicio con estado cambia de rol, por ejemplo, a principal o secundario. Las réplicas principales reciben el estado de escritura (se les permite crear y escribir en Reliable Collections). Las réplicas secundarias reciben el estado de lectura (solo pueden leer desde Reliable Collections existentes). La mayoría del trabajo en un servicio con estado se lleva a cabo en la réplica principal. Las réplicas secundarias pueden realizar validación de solo lectura, generación de informes, minería de datos u otros trabajos de solo lectura.

## <a name="next-steps"></a>Pasos siguientes
* [Introducción a Reliable Services](service-fabric-reliable-services-introduction.md)
* [Guía de inicio rápido de Reliable Services](service-fabric-reliable-services-quick-start-java.md)
