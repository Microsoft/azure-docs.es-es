---
title: Comunicación remota mediante Java en Azure Service Fabric
description: La comunicación remota de Service Fabric permite a los clientes y servicios comunicarse con los servicios Java mediante la llamada a procedimiento remoto.
author: PavanKunapareddyMSFT
ms.topic: conceptual
ms.date: 06/30/2017
ms.custom: devx-track-java
ms.author: pakunapa
ms.openlocfilehash: d53d20510db70d81aab796efab48de40c880bb3a
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87316131"
---
# <a name="service-remoting-in-java-with-reliable-services"></a>Comunicación remota en Java con Reliable Services
> [!div class="op_single_selector"]
> * [C# en Windows](service-fabric-reliable-services-communication-remoting.md)
> * [Java en Linux](service-fabric-reliable-services-communication-remoting-java.md)
>
>

Para los servicios que no están vinculados a una pila o un protocolo de comunicación concretos, como WebAPI, Windows Communication Foundation (WCF) u otros, el marco de trabajo de Reliable Services ofrece un mecanismo de comunicación remota para configurar de manera rápida y sencilla llamadas a procedimiento remoto para servicios.  En este artículo se describe cómo configurar llamadas a procedimiento remoto para servicios que se escriben con Java.

## <a name="set-up-remoting-on-a-service"></a>Configurar la comunicación remota en un servicio
La configuración de la comunicación remota para un servicio se realiza en dos sencillos pasos:

1. Cree una interfaz para que la implemente su servicio. Esta interfaz define los métodos que están disponibles para la llamada a procedimiento remoto en el servicio. Los métodos deben ser métodos asincrónicos que devuelven tareas. La interfaz debe implementar `microsoft.serviceFabric.services.remoting.Service` para indicar que el servicio tiene una interfaz de comunicación remota.
2. Utilice un agente de escucha de comunicación remota en su servicio. Se trata de una implementación de `CommunicationListener` que ofrece capacidades de comunicación remota. `FabricTransportServiceRemotingListener` puede utilizarse para crear un agente de escucha de comunicación remota con el protocolo de transporte de comunicación remota predeterminado.

Por ejemplo, el siguiente servicio sin estado expone un método único para obtener "Hello World" a través de la llamada a procedimiento remoto:

```java
import java.util.ArrayList;
import java.util.concurrent.CompletableFuture;
import java.util.List;
import microsoft.servicefabric.services.communication.runtime.ServiceInstanceListener;
import microsoft.servicefabric.services.remoting.Service;
import microsoft.servicefabric.services.runtime.StatelessService;

public interface MyService extends Service {
    CompletableFuture<String> helloWorldAsync();
}

class MyServiceImpl extends StatelessService implements MyService {
    public MyServiceImpl(StatelessServiceContext context) {
       super(context);
    }

    public CompletableFuture<String> helloWorldAsync() {
        return CompletableFuture.completedFuture("Hello!");
    }

    @Override
    protected List<ServiceInstanceListener> createServiceInstanceListeners() {
        ArrayList<ServiceInstanceListener> listeners = new ArrayList<>();
        listeners.add(new ServiceInstanceListener((context) -> {
            return new FabricTransportServiceRemotingListener(context,this);
        }));
        return listeners;
    }
}
```

> [!NOTE]
> Los argumentos y los tipos devueltos en la interfaz de servicio pueden ser cualquier tipo simple, complejo o personalizado, pero deben ser serializables.
>
>

## <a name="call-remote-service-methods"></a>Llamar a métodos de servicio remoto
La llamada a métodos en un servicio con la pila de comunicación remota se realiza mediante un proxy local al servicio a través de la clase `microsoft.serviceFabric.services.remoting.client.ServiceProxyBase` . El método `ServiceProxyBase` crea un proxy local con la misma interfaz que implementa el servicio. Con ese proxy, puede llamar simplemente a los métodos en la interfaz de forma remota.

```java

MyService helloWorldClient = ServiceProxyBase.create(MyService.class, new URI("fabric:/MyApplication/MyHelloWorldService"));

CompletableFuture<String> message = helloWorldClient.helloWorldAsync();

```

El marco de trabajo de comunicación remota propaga las excepciones generadas en el servicio al cliente. Por lo tanto, la lógica de control de excepciones en el cliente mediante `ServiceProxyBase` puede controlar excepciones directamente que el servicio genera.

## <a name="service-proxy-lifetime"></a>Duración del proxy de servicio
La creación de ServiceProxy es una operación ligera, por lo que se puede crear todas las veces que se necesite. Las instancias de ServiceProxy pueden volver a usarse siempre que se necesiten. Si una llamada a procedimiento remoto inicia una excepción, se puede reutilizar la misma instancia de proxy. Cada ServiceProxy contiene un cliente de comunicación que se usa para enviar mensajes a través de la conexión. Mientras se invocan llamadas remotas, se realizan comprobaciones internas para determinar si el cliente de comunicación es válido. En función de los resultados de esas comprobaciones, el cliente de comunicación se vuelve a crear si es necesario. Por lo tanto, si se produce una excepción, no es necesario volver a crear `ServiceProxy`.

### <a name="serviceproxyfactory-lifetime"></a>Duración de ServiceProxyFactory
[FabricServiceProxyFactory](/java/api/microsoft.servicefabric.services.remoting.client.fabricserviceproxyfactory) es un tejido que crea un proxy para distintas interfaces remotas. Si usa la API `ServiceProxyBase.create` para crear el proxy, el marco crea un `FabricServiceProxyFactory`.
Resulta útil crear uno manualmente cuando necesite invalidar las propiedades [IServiceRemotingClientFactory](/java/api/microsoft.servicefabric.services.remoting.client.serviceremotingclientfactory).
Factory es una operación costosa. `FabricServiceProxyFactory` mantiene la memoria caché de los clientes de comunicación.
El procedimiento recomendado consiste en almacenar `FabricServiceProxyFactory` en caché lo más posible.

## <a name="remoting-exception-handling"></a>Control de excepciones remota
Toda la excepción remota generada por la API de servicio se vuelve a enviar al cliente como RuntimeException o como FabricException.

ServiceProxy controla todas las excepciones de conmutación por error para la partición de servicio para la que se crea. Vuelve a resolver los puntos de conexión si existen excepciones de conmutación por error (excepciones no transitorias) y recupera la llamada con el punto de conexión correcto. El número de reintentos para la excepción de conmutación por error es indefinido.
En el caso de TransientExceptions, solo recupera la llamada.

[OperationRetrySettings](/java/api/microsoft.servicefabric.services.communication.client.operationretrysettings) proporciona los parámetros de reintento predeterminados.
Se pueden configurar estos valores pasando el objeto OperationRetrySettings al constructor ServiceProxyFactory.

## <a name="next-steps"></a>Pasos siguientes
* [Protección de la comunicación para Reliable Services](service-fabric-reliable-services-secure-communication-java.md)
