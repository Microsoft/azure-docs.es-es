---
title: Pruebas unitarias para servicios con estado en Azure Service Fabric | Microsoft Docs
description: Obtenga información sobre los conceptos y procedimientos recomendados para las pruebas unitarias en servicios con estado de Service Fabric.
services: service-fabric
documentationcenter: .net
author: athinanthny
manager: chackdan
editor: vturecek
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 09/04/2018
ms.author: atsenthi
ms.openlocfilehash: ad7cf3a1dfcef8795ceb378a59a1cf0b2010293e
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65595500"
---
# <a name="unit-testing-stateful-services-in-service-fabric"></a>Prueba unitaria de servicios con estado en Service Fabric

En este artículo se cubren los conceptos y procedimientos recomendados para las pruebas unitarias en servicios con estado de Service Fabric. Las pruebas unitarias dentro de Service Fabric necesitan sus propias consideraciones ya que el código de la aplicación se ejecuta activamente en varios contextos diferentes. En este artículo se describen las prácticas utilizadas para garantizar que el código de aplicación está contemplado en cada uno de los diferentes contextos en que se puede ejecutar.

## <a name="unit-testing-and-mocking"></a>Prueba unitaria y de simulación
Las pruebas unitarias en el contexto de este artículo son las pruebas automatizadas que pueden ejecutarse en el contexto de un ejecutor de pruebas como NUnit o MSTest. Las pruebas unitarias dentro de este artículo no realizan operaciones en un recurso remoto, como una base de datos o la API de RESTful. Estos recursos remotos deben simularse. La simulación en el contexto de este artículo es imitar, registrar y controlar los valores devueltos para los recursos remotos.

### <a name="service-fabric-considerations"></a>Consideraciones de Service Fabric
Realizar una prueba unitaria a un servicio con estado de Service Fabric requiere varias consideraciones. En primer lugar, el código de servicio se ejecuta en varios nodos, pero con distintos roles. Las pruebas unitarias deben evaluar el código sometido a cada rol para lograr una cobertura completa. Los distintos roles son principal, secundaria activa, secundaria inactiva y desconocido. El rol Ninguno normalmente no requiere de ninguna cobertura especial, ya que Service Fabric lo considera un servicio nulo o null. En segundo lugar, cada nodo cambiará su rol en un momento dado. Para lograr una cobertura completa, la ruta de ejecución del código se debe probar mientras se producen los cambios de rol.

## <a name="why-unit-test-stateful-services"></a>¿Por qué realizar pruebas unitarias de servicios con estado? 
Las pruebas unitarias para los servicios con estado pueden ayudar a detectar errores comunes que no necesariamente detectaría una prueba unitaria convencional específica para dominios o aplicaciones. Por ejemplo, si el servicio con estado tiene cualquier estado en memoria, este tipo de prueba puede comprobar si este estado en memoria se mantiene sincronizado entre las réplicas. Este tipo de pruebas también puede comprobar que un servicio con estado responde correctamente a los tokens de cancelación pasados por la orquestación de Service Fabric. Cuando se desencadenan las cancelaciones, el servicio debe detener las operaciones de larga ejecución y las asincrónicas.  

## <a name="common-practices"></a>Prácticas comunes

La siguiente sección le da consejos sobre las prácticas más comunes para realizar pruebas unitarias a un servicio con estado. También aconseja sobre lo que debe tener una capa de simulación para adaptarse lo más posible a la orquestación de Service Fabric y la administración de estados. [ServiceFabric.Mocks](https://www.nuget.org/packages/ServiceFabric.Mocks/) a partir de la versión 3.3.0 o posterior es una de dichas bibliotecas que proporcionan la funcionalidad de simulación recomendada y siguen las prácticas que se describen a continuación.

### <a name="arrangement"></a>Organización

#### <a name="use-multiple-service-instances"></a>Uso de varias instancias de servicio
Las pruebas unitarias deben ejecutar varias instancias de un servicio con estado. Esto simula lo que sucede realmente en el clúster cuando Service Fabric aprovisiona varias réplicas que ejecutan el servicio entre los distintos nodos. Sin embargo, cada una de estas instancias se va a ejecutar en un contexto diferente. Cuando se ejecuta la prueba, cada instancia debe estar preparada con la configuración de roles que se espera en el clúster. Por ejemplo, si el servicio debe tener un tamaño de réplica de destino de 3, Service Fabric aprovisionará tres réplicas en distintos nodos. Uno de las cuales es la principal y las otras dos son secundarias activas.

En la mayoría de los casos, la ruta de ejecución del servicio puede variar ligeramente para cada uno de estos roles. Por ejemplo, si el servicio no debe aceptar las solicitudes de una secundaria activa, el servicio puede tener una comprobación para que este caso devuelva una excepción informativa indicando que se intentó hacer una solicitud en una réplica secundaria. Tener varias instancias permitirá que se pruebe esta situación.

Además, tener varias instancias permite que las pruebas cambien los roles de cada una de estas instancias para comprobar que las respuestas son coherentes a pesar de los cambios de rol.

#### <a name="mock-the-state-manager"></a>Simulación del administrador de estados
El administrador de estados debe considerarse como un recurso remoto y, por lo tanto, se debe simular. Cuando se hace una simulación del administrador de estados, debe haber algún almacenamiento subyacente en memoria para el seguimiento de lo que se guarda en el administrador de estados, de forma que pueda leerse y comprobarse. Una manera sencilla de conseguirlo es crear instancias ficticias de cada uno de los tipos de colecciones de confianza. Dentro de esos objetos ficticios, utilice un tipo de datos que se alinee estrechamente con las operaciones realizadas en esa colección. Los siguientes son algunos tipos de datos sugeridos para cada colección de confianza

- IReliableDictionary < TKey, TValue > -> System.Collections.Concurrent.ConcurrentDictionary < TKey, TValue >
- IReliableQueue<T> -> System.Collections.Generic.Queue<T>
- IReliableConcurrentQueue<T> -> System.Collections.Concurrent.ConcurrentQueue<T>

#### <a name="many-state-manager-instances-single-storage"></a>Muchas instancias de administrador de estados, un almacenamiento único
Como se mencionó antes, el administrador de estados y las colecciones de confianza deben tratarse como un recurso remoto. Por lo tanto, estos recursos deben y van a simularse dentro de las pruebas unitarias. Sin embargo, cuando se ejecutan varias instancias de un servicio con estado, es un desafío mantener cada administrador de estado ficticio sincronizado entre las distintas instancias de servicio con estado. Cuando se ejecuta el servicio con estado en el clúster, Service Fabric se encarga de mantener el administrador de estado de cada réplica secundaria coherente con la réplica principal. Por lo tanto, las pruebas se deben comportar del mismo modo para que puedan simular los cambios de rol.

Una manera sencilla de lograr esta sincronización es usar un patrón de singleton para el objeto subyacente que almacena los datos escritos en cada colección de confianza. Por ejemplo, si un servicio con estado está usando un `IReliableDictionary<string, string>`. El administrador de estado ficticio debe devolver una simulación de `IReliableDictionary<string, string>`. Esa simulación puede usar un `ConcurrentDictionary<string, string>` para realizar un seguimiento de los pares escritos de clave y valor. El `ConcurrentDictionary<string, string>` debe ser un singleton que usen todas las instancias de administradores de estados que se pasan al servicio.

#### <a name="keep-track-of-cancellation-tokens"></a>Realización de un seguimiento de los tokens de cancelación
Los tokens de cancelación son un aspecto importante de los servicios con estado, aunque a menudo se pasan por alto. Cuando Service Fabric inicia una réplica principal para un servicio con estado, se proporciona un token de cancelación. Este token de cancelación está pensado para indicar al servicio cuando se esté quitando o degradando a un rol diferente. El servicio con estado debe detener las operaciones de larga ejecución o asincrónicas para que Service Fabric puede completar el flujo de trabajo del cambio de rol.

Al ejecutar las pruebas unitarias, los tokens de cancelación que se proporcionan a RunAsync, ChangeRoleAsync, OpenAsync y CloseAsync deben mantenerse durante la ejecución de pruebas. Conservar estos tokens permitirá que la prueba simule el cierre o degradación del servicio y demostrará que el servicio responde adecuadamente.

#### <a name="test-end-to-end-with-mocked-remote-resources"></a>Prueba de un extremo a otro con recursos remotos simulados
Las pruebas unitarias deben ejecutar la mayor cantidad posible del código de aplicación que puede modificar el estado del servicio con estado. Se recomienda que las pruebas sean más bien de un extremo a otro. Las únicas simulaciones que existen son para grabar, simular y comprobar las interacciones del recurso remoto. Esto incluye las interacciones con el administrador de estados y las colecciones de confianza. El fragmento de código siguiente es un ejemplo de gherkin para una prueba que muestra la realización de pruebas de un extremo a otro:

```
    Given stateful service named "fabric:/MyApp/MyService" is created
    And a new replica is created as "Primary" with id "111"
    And a new replica is created as "IdleSecondary" with id "222"
    And a new replica is created as "IdleSecondary" with id "333"
    And all idle secondary replicas are promoted to active secondary
    When a request is made to add the an employee "John Smith"
    And the active secondary replica "222" is promoted to primary
    And a request is made to get all employees
    Then the request should should return the "John Smith" employee
```

Esta prueba valida que los datos que están capturándose en una réplica están disponibles para una réplica secundaria cuando se promueve a principal. Suponiendo que una colección de confianza es la memoria auxiliar de los datos de empleados, un posible error que se puede detectar con esta prueba es si el código de aplicación no ha ejecutado `CommitAsync` en la transacción para guardar al empleado nuevo. En ese caso, la segunda solicitud para obtener a los empleados no devolvería al empleado que se agregó con la primera solicitud.

### <a name="acting"></a>Acción
#### <a name="mimic-service-fabric-replica-orchestration"></a>Imitación de la orquestación de réplica de Service Fabric
Al administrar varias instancias de servicio, las pruebas deben inicializar y anular los servicios de la misma forma que la orquestación de Service Fabric. Por ejemplo, cuando se cree un servicio en una nueva réplica principal, Service Fabric invocará a CreateServiceReplicaListener, OpenAsync, ChangeRoleAsync y RunAsync. Los eventos de ciclo de vida se documentan en los siguientes artículos:

- [Inicio de un servicio con estado](service-fabric-reliable-services-lifecycle.md#stateful-service-startup)
- [Cierre de un servicio con estado](service-fabric-reliable-services-lifecycle.md#stateful-service-shutdown)
- [Intercambios de servicios con estado principales](service-fabric-reliable-services-lifecycle.md#stateful-service-primary-swaps)

#### <a name="run-replica-role-changes"></a>Ejecución de los cambios de rol de réplica
Las pruebas unitarias deben cambiar los roles de las instancias de servicio de la misma forma que la orquestación de Service Fabric. La máquina de estados de rol está documentada en el siguiente artículo:

[Máquina de estados de rol de réplica](service-fabric-concepts-replica-lifecycle.md#replica-role)

Simular cambios de rol es uno de los aspectos más críticos de las pruebas y puede revelar problemas en los que los estados de réplica no son coherentes entre sí. Los estados de réplica incoherentes pueden deberse a que el estado en memoria se almacena en variables de instancia a nivel de clase o estático. Un posible ejemplo pueden ser los tokens de cancelación, las enumeraciones y los objetos o valores de configuración. Esto también garantiza que el servicio está respetando los tokens de cancelación proporcionados durante RunAsync para permitir que se produzca el cambio de rol. Simular los cambios de rol también puede revelar problemas que posiblemente surgen si no se escribe código para permitir que RunAsync se invoque varias veces.

#### <a name="cancel-cancellation-tokens"></a>Cancelación de los tokens de cancelación
Debe haber pruebas unitarias en las que el token de cancelación proporcionado a RunAsync se cancele. Esto permitirá que la prueba compruebe el cierre discreto del servicio. Durante el cierre, cualquier operación de larga ejecución o asincrónica debe detenerse. Un ejemplo de proceso de larga ejecución que puede existir en un servicio es el que escucha los mensajes en una cola de confianza. Puede que exista directamente en RunAsync o en un subproceso en segundo plano. La implementación debe incluir una lógica para salir de la operación si se cancela este token de cancelación.

Si los servicios con estado hacen uso de cualquier estado en memoria o en caché que solo debe existir en la réplica principal, se debe desechar en este momento. Esto es para asegurarse de que este estado es coherente incluso si, más tarde, el nodo se vuelve principal de nuevo. Las pruebas de cancelación permitirán que la prueba compruebe que el estado se elimina correctamente.

#### <a name="execute-requests-against-multiple-replicas"></a>Ejecución de solicitudes en varias réplicas
Las pruebas de aserción deben ejecutar la misma solicitud en otras réplicas. Cuando se emparejen con los cambios de rol, se podrán descubrir problemas de coherencia. Es posible que una prueba de ejemplo realice los siguientes pasos:
1. Ejecutar una solicitud de escritura en la réplica principal actual
2. Ejecutar una solicitud de lectura que devuelva los datos escritos en el paso 1 en la réplica principal actual
3. Promover una secundaria a principal. Esto también debe degradar la réplica principal actual a secundaria
4. Ejecutar la misma solicitud de lectura del paso 2 en la nueva secundaria.

En el último paso, la prueba puede validar si los datos devueltos son coherentes. Un problema potencial que podría descubrir es que los datos devueltos por el servicio pueden estar en memoria pero, en última instancia, con el respaldo de una colección de confianza. Es posible que esos datos en memoria no se mantengan sincronizados correctamente con lo que existe en la colección de confianza.

Los datos en memoria se utilizan normalmente para crear índices secundarios o agregaciones de datos que existen en una colección de confianza.

### <a name="asserting"></a>Aserción
#### <a name="ensure-responses-match-across-replicas"></a>Asegúrese de que las respuestas entre réplicas coinciden
Las pruebas unitarias deben afirmar que la respuesta para una solicitud determinada sea coherente en varias réplicas después de realizar la transición a principal. Esto puede evidenciar posibles problemas en los que los datos proporcionados en la respuesta no estén respaldados por una colección de confianza o estén almacenados en memoria sin un mecanismo para sincronizar los datos entre réplicas. Así se asegurará de que el servicio envía de regreso respuestas coherentes después de que Service Fabric vuelva a equilibrar o conmute por error a una nueva réplica principal.

#### <a name="verify-service-respects-cancellation"></a>Comprobación de la cancelación de aspectos del servicio
Se debe comprobar que los procesos asincrónicos o de larga ejecución que deben finalizar cuando se cancela un token de cancelación realmente se finalicen después de la cancelación. Así se asegurará de que, a pesar del cambio de roles de la réplica, los procesos que no están pensados para seguirse ejecutando en las réplicas no principales se detendrán antes de que se complete la transición. También se pueden revelar problemas donde este proceso evita que se complete una solicitud de cambio de rol o de cierre desde Service Fabric.

#### <a name="verify-which-replicas-should-serve-requests"></a>Comprobación de qué réplicas deben atender las solicitudes
Las pruebas deben validar el comportamiento esperado si se enruta una solicitud a una réplica no principal. Service Fabric proporciona la capacidad de que las réplicas secundarias atiendan las solicitudes. Sin embargo, las escrituras en colecciones de confianza solo pueden realizarse desde la réplica principal. Si la aplicación pretende que solo las réplicas principales atiendan las solicitudes, o bien solo un subconjunto de las solicitudes puede controlarse mediante una réplica secundaria, las pruebas deben validar el comportamiento esperado para los casos positivos y negativos. El caso negativo sería que una solicitud se enrute a una réplica que no debería encargarse de la solicitud, y el positivo sería lo opuesto.

## <a name="next-steps"></a>Pasos siguientes
Aprenda cómo [realizar pruebas unitarias de servicios con estado](service-fabric-how-to-unit-test-stateful-services.md).
