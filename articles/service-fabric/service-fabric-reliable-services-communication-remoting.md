---
title: Comunicación remota del servicio mediante C# en Service Fabric
description: La comunicación remota de Service Fabric permite a los clientes y servicios comunicarse con los servicios de C# mediante la llamada a procedimiento remoto.
author: vturecek
ms.topic: conceptual
ms.date: 09/20/2017
ms.author: vturecek
ms.custom: devx-track-csharp
ms.openlocfilehash: 246b1456c05605c4015c19e1a139e9ad65f6eaba
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89022163"
---
# <a name="service-remoting-in-c-with-reliable-services"></a>Comunicación remota del servicio en C# con Reliable Services

> [!div class="op_single_selector"]
> * [C# en Windows](service-fabric-reliable-services-communication-remoting.md)
> * [Java en Linux](service-fabric-reliable-services-communication-remoting-java.md)
>
>

Para los servicios que no están vinculados a una pila o un protocolo de comunicación concretos, como API web, Windows Communication Foundation (WCF) u otros, el marco de trabajo de Reliable Services ofrece un mecanismo de comunicación remota para configurar de manera rápida y sencilla llamadas a procedimiento remoto para servicios. En este artículo se describe la manera de configurar llamadas a procedimiento remoto para servicios que se escriben con C#.

## <a name="set-up-remoting-on-a-service"></a>Configurar la comunicación remota en un servicio

Puede configurar la comunicación remota de un servicio en dos sencillos pasos:

1. Cree una interfaz para que la implemente su servicio. Esta interfaz define los métodos que están disponibles para la llamada a procedimiento remoto en el servicio. Los métodos deben ser métodos asincrónicos que devuelven tareas. La interfaz debe implementar `Microsoft.ServiceFabric.Services.Remoting.IService` para indicar que el servicio tiene una interfaz de comunicación remota.
2. Utilice un agente de escucha de comunicación remota en su servicio. Un agente de escucha de comunicación remota es una implementación de `ICommunicationListener` que ofrece funcionalidades de comunicación remota. El espacio de nombres `Microsoft.ServiceFabric.Services.Remoting.Runtime` contiene un método de extensión, `CreateServiceRemotingInstanceListeners` para los servicios con y sin estado que puede usarse para crear un agente de escucha de comunicación remota mediante el protocolo de transporte de comunicación remota predeterminado.

>[!NOTE]
>El espacio de nombres `Remoting` está disponible como un paquete NuGet independiente denominado `Microsoft.ServiceFabric.Services.Remoting`.

Por ejemplo, el siguiente servicio sin estado expone un método único para obtener "Hello World" a través de la llamada a procedimiento remoto:

```csharp
using Microsoft.ServiceFabric.Services.Communication.Runtime;
using Microsoft.ServiceFabric.Services.Remoting;
using Microsoft.ServiceFabric.Services.Remoting.Runtime;
using Microsoft.ServiceFabric.Services.Runtime;

public interface IMyService : IService
{
    Task<string> HelloWorldAsync();
}

class MyService : StatelessService, IMyService
{
    public MyService(StatelessServiceContext context)
        : base (context)
    {
    }

    public Task<string> HelloWorldAsync()
    {
        return Task.FromResult("Hello!");
    }

    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
     return this.CreateServiceRemotingInstanceListeners();
    }
}
```

> [!NOTE]
> Los argumentos y los tipos devueltos de la interfaz de servicio pueden ser cualquier tipo simple, complejo o personalizado, pero deben poderse serializar mediante [DataContractSerializer](/dotnet/framework/wcf/feature-details/types-supported-by-the-data-contract-serializer) de .NET.
>
>

## <a name="call-remote-service-methods"></a>Llamar a métodos de servicio remoto

La llamada a métodos en un servicio con la pila de comunicación remota se realiza mediante un proxy local al servicio a través de la clase `Microsoft.ServiceFabric.Services.Remoting.Client.ServiceProxy` . El método `ServiceProxy` crea un proxy local con la misma interfaz que implementa el servicio. Con ese proxy, puede llamar a los métodos en la interfaz de forma remota.

```csharp

IMyService helloWorldClient = ServiceProxy.Create<IMyService>(new Uri("fabric:/MyApplication/MyHelloWorldService"));

string message = await helloWorldClient.HelloWorldAsync();

```

El marco de trabajo de comunicación remota propaga las excepciones generadas por el servicio al cliente. Como resultado, al usar `ServiceProxy`, el cliente es responsable de controlar las excepciones generadas por el servicio.

## <a name="service-proxy-lifetime"></a>Duración del proxy de servicio

La creación del proxy de servicio es una operación ligera, por lo que se puede crear todas las veces que se necesite. Las instancias de proxy de servicio pueden volver a usarse siempre que se necesiten. Si una llamada a procedimiento remoto inicia una excepción, se puede reutilizar la misma instancia de proxy. Cada proxy de servicio contiene un cliente de comunicación que se usa para enviar mensajes a través de la conexión. Mientras se invocan llamadas remotas, se realizan comprobaciones internas para determinar si el cliente de comunicación es válido. En función de los resultados de esas comprobaciones, el cliente de comunicación se vuelve a crear si es necesario. Por lo tanto, si se produce una excepción, no es necesario volver a crear `ServiceProxy`.

### <a name="service-proxy-factory-lifetime"></a>Duración de la fábrica de proxy de servicio

[ServiceProxyFactory](/dotnet/api/microsoft.servicefabric.services.remoting.client.serviceproxyfactory) es una fábrica que crea instancias de proxy para interfaces remotas diferentes. Si usa la API `ServiceProxyFactory.CreateServiceProxy` para crear un proxy, el marco de trabajo crea un proxy de servicio singleton.
Es útil crear uno manualmente cuando necesite invalidar las propiedades [IServiceRemotingClientFactory](/dotnet/api/microsoft.servicefabric.services.remoting.v1.client.iserviceremotingclientfactory).
La creación de fábricas es una operación costosa. Una fábrica de proxy de servicio mantiene una caché interna del cliente de comunicación.
Un procedimiento recomendado consiste en almacenar en caché la fábrica de proxy de servicio tanto como sea posible.

## <a name="remoting-exception-handling"></a>Control de excepciones remota

Todas las excepciones remotas generadas por la API de servicio se vuelven a enviar al cliente como AggregateException. Las excepciones remotas deben poderse serializar mediante DataContract. Si no, la API de proxy genera [ServiceException](/dotnet/api/microsoft.servicefabric.services.communication.serviceexception) con el error de serialización.

El proxy de servicio controla todas las excepciones de conmutación por error de la partición de servicio para la que se crea. Vuelve a resolver los puntos de conexión si existen excepciones de conmutación por error (excepciones no transitorias) y recupera la llamada con el punto de conexión correcto. El número de reintentos de las excepciones de conmutación por error es indefinido.
Si se producen excepciones transitorias, el proxy vuelve a intentar la llamada.

[OperationRetrySettings](/dotnet/api/microsoft.servicefabric.services.communication.client.operationretrysettings) proporciona los parámetros de reintento predeterminados.

El usuario puede configurar estos valores pasando el objeto OperationRetrySettings al constructor ServiceProxyFactory.

## <a name="use-the-remoting-v2-stack"></a>Uso de la pila de comunicación remota V2

A partir de la versión 2.8 del paquete de comunicación remota de NuGet, tiene la opción de usar la pila de comunicación remota V2. La pila de comunicación remota V2 funciona mejor. También proporciona características como serialización personalizada y API más conectables.
El código de plantilla continúa usando la pila de comunicación remota V1.
La comunicación remota V2 no es compatible con V1 (la pila de comunicación remota anterior). Siga las instrucciones del artículo [Actualización de V1 a V2](#upgrade-from-remoting-v1-to-remoting-v2) para evitar los efectos sobre la disponibilidad del servicio.

Los siguientes enfoques están disponibles para habilitar la pila V2.

### <a name="use-an-assembly-attribute-to-use-the-v2-stack"></a>Uso de un atributo de ensamblado para utilizar la pila V2

Estos pasos cambian el código de la plantilla para usar la pila V2 mediante un atributo de ensamblado.

1. Cambie el recurso del punto de conexión de `"ServiceEndpoint"` a `"ServiceEndpointV2"` en el manifiesto del servicio.

   ```xml
   <Resources>
    <Endpoints>
      <Endpoint Name="ServiceEndpointV2" />
    </Endpoints>
   </Resources>
   ```

2. Use el método de extensión `Microsoft.ServiceFabric.Services.Remoting.Runtime.CreateServiceRemotingInstanceListeners` para crear agentes de escucha remotos (esto es igual tanto para V1 como para V2).

   ```csharp
    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return this.CreateServiceRemotingInstanceListeners();
    }
   ```

3. Marque el ensamblado que contiene las interfaces de comunicación remota con un atributo `FabricTransportServiceRemotingProvider`.

   ```csharp
   [assembly: FabricTransportServiceRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2, RemotingClientVersion = RemotingClientVersion.V2)]
   ```

No se requiere ningún cambio en el código del proyecto del cliente.
Compile el ensamblado de cliente con el ensamblado de la interfaz para asegurarse de que se usa el atributo de ensamblado mostrado anteriormente.

### <a name="use-explicit-v2-classes-to-use-the-v2-stack"></a>Uso de clases V2 explícitas para utilizar la pila V2

Como alternativa al uso de un atributo de ensamblado, la pila V2 también se puede habilitar mediante clases V2 explícitas.

Estos pasos cambian el código de la plantilla para usar la pila V2 mediante las clases V2 explícitas.

1. Cambie el recurso del punto de conexión de `"ServiceEndpoint"` a `"ServiceEndpointV2"` en el manifiesto del servicio.

   ```xml
   <Resources>
    <Endpoints>
      <Endpoint Name="ServiceEndpointV2" />
    </Endpoints>
   </Resources>
   ```

2. Use [FabricTransportServiceRemotingListener](/dotnet/api/microsoft.servicefabric.services.remoting.v2.fabrictransport.runtime.fabrictransportserviceremotinglistener?view=azure-dotnet) del espacio de nombres `Microsoft.ServiceFabric.Services.Remoting.V2.FabricTransport.Runtime`.

   ```csharp
   protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return new[]
        {
            new ServiceInstanceListener((c) =>
            {
                return new FabricTransportServiceRemotingListener(c, this);

            })
        };
    }
   ```

3. Use [FabricTransportServiceRemotingClientFactory ](/dotnet/api/microsoft.servicefabric.services.remoting.v2.fabrictransport.client.fabrictransportserviceremotingclientfactory?view=azure-dotnet) del espacio de nombres `Microsoft.ServiceFabric.Services.Remoting.V2.FabricTransport.Client` para crear clientes.

   ```csharp
   var proxyFactory = new ServiceProxyFactory((c) =>
          {
              return new FabricTransportServiceRemotingClientFactory();
          });
   ```

## <a name="upgrade-from-remoting-v1-to-remoting-v2"></a>Actualización de la comunicación remota V1 a la comunicación remota V2

Para actualizar de V1 a V2, se requieren actualizaciones en dos pasos. Siga los pasos de esta secuencia.

1. Actualice el servicio V1 al servicio V2 mediante este atributo.
Este cambio garantiza que el servicio escucha en el agente de escucha V1 y V2.

    a. Agregue un recurso de punto de conexión con el nombre "ServiceEndpointV2" en el manifiesto de servicio.
      ```xml
      <Resources>
        <Endpoints>
          <Endpoint Name="ServiceEndpointV2" />  
        </Endpoints>
      </Resources>
      ```

    b. Use el método de extensión siguiente para crear un agente de escucha de comunicación remota.

    ```csharp
    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return this.CreateServiceRemotingInstanceListeners();
    }
    ```

    c. Agregue un atributo de ensamblado en las interfaces de comunicación remota para usar el agente de escucha V1 y V2, y el cliente V2.
    ```csharp
    [assembly: FabricTransportServiceRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2|RemotingListenerVersion.V1, RemotingClientVersion = RemotingClientVersion.V2)]

      ```
2. Actualice el cliente V1 a un cliente V2 mediante el atributo de cliente V2.
Este paso garantiza que el cliente usa la pila V2.
No se requiere ningún cambio en el proyecto o servicio de cliente. Basta con compilar los proyectos de cliente con el ensamblado de interfaz actualizado.

3. Este paso es opcional. Use el atributo de agente de escucha V2 y, a continuación, actualice el servicio V2.
Este paso garantiza que el servicio escucha solo en el agente de escucha V2.

    ```csharp
    [assembly: FabricTransportServiceRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2, RemotingClientVersion = RemotingClientVersion.V2)]
    ```


## <a name="use-the-remoting-v2-interface-compatible-stack"></a>Uso de la pila de comunicación remota V2 (compatible con interfaz)

 La pila de comunicación remota V2 (compatible con interfaz, conocida como V2_1) tiene todas las características de la pila de comunicación remota V2. Su pila de interfaz es compatible con la pila de comunicación remota V1, pero no es compatible con V2 y V1. Para actualizar de V1 a V2_1 sin que la disponibilidad del servicio se vea afectada, siga los pasos descritos en el artículo Actualización de V1 a V2 (compatible con interfaz).


### <a name="use-an-assembly-attribute-to-use-the-remoting-v2-interface-compatible-stack"></a>Uso de un atributo de ensamblado para utilizar la pila de comunicación remota V2 (compatible con interfaz)

Siga estos pasos para cambiar a una pila V2_1.

1. Agregue un recurso de punto de conexión con el nombre "ServiceEndpointV2_1" en el manifiesto de servicio.

   ```xml
   <Resources>
    <Endpoints>
      <Endpoint Name="ServiceEndpointV2_1" />  
    </Endpoints>
   </Resources>
   ```

2. Use el método de extensión de comunicación remota para crear un agente de escucha de comunicación remota.

   ```csharp
    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return this.CreateServiceRemotingInstanceListeners();
    }
   ```

3. Agregue un [atributo de ensamblado](/dotnet/api/microsoft.servicefabric.services.remoting.fabrictransport.fabrictransportserviceremotingproviderattribute?view=azure-dotnet) en las interfaces de comunicación remota.

   ```csharp
    [assembly:  FabricTransportServiceRemotingProvider(RemotingListenerVersion=  RemotingListenerVersion.V2_1, RemotingClientVersion= RemotingClientVersion.V2_1)]

   ```

No se requiere ningún cambio en el proyecto del cliente.
Compile el ensamblado de cliente con el ensamblado de interfaz para asegurarse de que se usa el atributo de ensamblado anterior.

### <a name="use-explicit-remoting-classes-to-create-a-listenerclient-factory-for-the-v2-interface-compatible-version"></a>Uso de clases de comunicación remota para crear un agente de escucha o fábrica de cliente para la versión V2 (compatible con interfaz)

Siga estos pasos:

1. Agregue un recurso de punto de conexión con el nombre "ServiceEndpointV2_1" en el manifiesto de servicio.

   ```xml
   <Resources>
    <Endpoints>
      <Endpoint Name="ServiceEndpointV2_1" />  
    </Endpoints>
   </Resources>
   ```

2. Use el [agente de escucha de comunicación remota V2](/dotnet/api/microsoft.servicefabric.services.remoting.v2.fabrictransport.runtime.fabrictransportserviceremotinglistener?view=azure-dotnet). El nombre de recurso del punto de conexión de servicio predeterminado usado es "ServiceEndpointV2_1". Se debe definir en el manifiesto de servicio.

   ```csharp
   protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return new[]
        {
            new ServiceInstanceListener((c) =>
            {
                var settings = new FabricTransportRemotingListenerSettings();
                settings.UseWrappedMessage = true;
                return new FabricTransportServiceRemotingListener(c, this,settings);

            })
        };
    }
   ```

3. Use la [fábrica de cliente](/dotnet/api/microsoft.servicefabric.services.remoting.v2.fabrictransport.client.fabrictransportserviceremotingclientfactory?view=azure-dotnet) V2.
   ```csharp
   var proxyFactory = new ServiceProxyFactory((c) =>
          {
            var settings = new FabricTransportRemotingSettings();
            settings.UseWrappedMessage = true;
            return new FabricTransportServiceRemotingClientFactory(settings);
          });
   ```

## <a name="upgrade-from-remoting-v1-to-remoting-v2-interface-compatible"></a>Actualización de la comunicación remota V1 a la comunicación remota V2 (compatible con interfaz)

Para actualizar de V1 a V2 (compatible con interfaz, conocido como V2_1), se requieren actualizaciones en dos pasos. Siga los pasos de esta secuencia.

> [!NOTE]
> Al actualizar de la versión 1 a la versión 2, asegúrese de que el espacio de nombres `Remoting` se actualiza para usar la versión 2. Ejemplo: 'Microsoft.ServiceFabric.Services.Remoting.V2.FabricTransport.Client`
>
>

1. Actualice el servicio V1 al servicio V2_1 mediante el atributo siguiente.
Este cambio garantiza que el servicio escucha en el agente de escucha V1 y V2_1.

    a. Agregue un recurso de punto de conexión con el nombre "ServiceEndpointV2_1" en el manifiesto de servicio.
      ```xml
      <Resources>
        <Endpoints>
          <Endpoint Name="ServiceEndpointV2_1" />  
        </Endpoints>
      </Resources>
      ```

    b. Use el método de extensión siguiente para crear un agente de escucha de comunicación remota.

    ```csharp
    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return this.CreateServiceRemotingInstanceListeners();
    }
    ```

    c. Agregue el atributo de ensamblado en las interfaces de comunicación remota para usar el agente de escucha V1 y V2_1, y el cliente V2_1.
    ```csharp
   [assembly: FabricTransportServiceRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2_1 | RemotingListenerVersion.V1, RemotingClientVersion = RemotingClientVersion.V2_1)]

      ```
2. Actualice el cliente V1 al cliente V2_1 mediante el atributo de cliente V2_1.
Este paso garantiza que el cliente usa la pila V2_1.
No se requiere ningún cambio en el proyecto o servicio de cliente. Basta con compilar los proyectos de cliente con el ensamblado de interfaz actualizado.

3. Este paso es opcional. Quite la versión del agente de escucha V1 del atributo y, luego, actualice el servicio V2.
Este paso garantiza que el servicio escucha solo en el agente de escucha V2.

    ```csharp
    [assembly: FabricTransportServiceRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2_1, RemotingClientVersion = RemotingClientVersion.V2_1)]
    ```
  
### <a name="use-custom-serialization-with-a-remoting-wrapped-message"></a>Uso de la serialización personalizada con un mensaje encapsulado de comunicación remota

En los mensajes encapsulados de comunicación remota, se crea un único objeto encapsulado con todos los parámetros como campo.
Siga estos pasos:

1. Implemente la interfaz `IServiceRemotingMessageSerializationProvider` para proporcionar implementación para la serialización personalizada.
    Este fragmento de código muestra el aspecto de la implementación.

      ```csharp
      public class ServiceRemotingJsonSerializationProvider : IServiceRemotingMessageSerializationProvider
      {
        public IServiceRemotingMessageBodyFactory CreateMessageBodyFactory()
        {
          return new JsonMessageFactory();
        }

        public IServiceRemotingRequestMessageBodySerializer CreateRequestMessageSerializer(Type serviceInterfaceType, IEnumerable<Type> requestWrappedType, IEnumerable<Type> requestBodyTypes = null)
        {
          return new ServiceRemotingRequestJsonMessageBodySerializer();
        }

        public IServiceRemotingResponseMessageBodySerializer CreateResponseMessageSerializer(Type serviceInterfaceType, IEnumerable<Type> responseWrappedType, IEnumerable<Type> responseBodyTypes = null)
        {
          return new ServiceRemotingResponseJsonMessageBodySerializer();
        }
      }
      ```
      ```csharp
        class JsonMessageFactory : IServiceRemotingMessageBodyFactory
            {

              public IServiceRemotingRequestMessageBody CreateRequest(string interfaceName, string methodName, int numberOfParameters, object wrappedRequestObject)
              {
                return new JsonBody(wrappedRequestObject);
              }

              public IServiceRemotingResponseMessageBody CreateResponse(string interfaceName, string methodName, object wrappedRequestObject)
              {
                return new JsonBody(wrappedRequestObject);
              }
            }
      ```
      ```csharp
      class ServiceRemotingRequestJsonMessageBodySerializer : IServiceRemotingRequestMessageBodySerializer
        {
            private JsonSerializer serializer;

            public ServiceRemotingRequestJsonMessageBodySerializer()
            {
              serializer = JsonSerializer.Create(new JsonSerializerSettings()
              {
                TypeNameHandling = TypeNameHandling.All
                });
              }

              public IOutgoingMessageBody Serialize(IServiceRemotingRequestMessageBody serviceRemotingRequestMessageBody)
             {
               if (serviceRemotingRequestMessageBody == null)
               {
                 return null;
               }          
               using (var writeStream = new MemoryStream())
               {
                 using (var jsonWriter = new JsonTextWriter(new StreamWriter(writeStream)))
                 {
                   serializer.Serialize(jsonWriter, serviceRemotingRequestMessageBody);
                   jsonWriter.Flush();
                   var bytes = writeStream.ToArray();
                   var segment = new ArraySegment<byte>(bytes);
                   var segments = new List<ArraySegment<byte>> { segment };
                   return new OutgoingMessageBody(segments);
                 }
               }
              }

              public IServiceRemotingRequestMessageBody Deserialize(IIncomingMessageBody messageBody)
             {
               using (var sr = new StreamReader(messageBody.GetReceivedBuffer()))
               {
                 using (JsonReader reader = new JsonTextReader(sr))
                 {
                   var ob = serializer.Deserialize<JsonBody>(reader);
                   return ob;
                 }
               }
             }
            }
      ```
      ```csharp
      class ServiceRemotingResponseJsonMessageBodySerializer : IServiceRemotingResponseMessageBodySerializer
       {
         private JsonSerializer serializer;

        public ServiceRemotingResponseJsonMessageBodySerializer()
        {
          serializer = JsonSerializer.Create(new JsonSerializerSettings()
          {
              TypeNameHandling = TypeNameHandling.All
            });
          }

          public IOutgoingMessageBody Serialize(IServiceRemotingResponseMessageBody responseMessageBody)
          {
            if (responseMessageBody == null)
            {
              return null;
            }

            using (var writeStream = new MemoryStream())
            {
              using (var jsonWriter = new JsonTextWriter(new StreamWriter(writeStream)))
              {
                serializer.Serialize(jsonWriter, responseMessageBody);
                jsonWriter.Flush();
                var bytes = writeStream.ToArray();
                var segment = new ArraySegment<byte>(bytes);
                var segments = new List<ArraySegment<byte>> { segment };
                return new OutgoingMessageBody(segments);
              }
            }
          }

          public IServiceRemotingResponseMessageBody Deserialize(IIncomingMessageBody messageBody)
          {

             using (var sr = new StreamReader(messageBody.GetReceivedBuffer()))
             {
               using (var reader = new JsonTextReader(sr))
               {
                 var obj = serializer.Deserialize<JsonBody>(reader);
                 return obj;
               }
             }
           }
       }
    ```
    ```csharp
    class JsonBody : WrappedMessage, IServiceRemotingRequestMessageBody, IServiceRemotingResponseMessageBody
    {
          public JsonBody(object wrapped)
          {
            this.Value = wrapped;
          }

          public void SetParameter(int position, string parameName, object parameter)
          {  //Not Needed if you are using WrappedMessage
            throw new NotImplementedException();
          }

          public object GetParameter(int position, string parameName, Type paramType)
          {
            //Not Needed if you are using WrappedMessage
            throw new NotImplementedException();
          }

          public void Set(object response)
          { //Not Needed if you are using WrappedMessage
            throw new NotImplementedException();
          }

          public object Get(Type paramType)
          {  //Not Needed if you are using WrappedMessage
            throw new NotImplementedException();
          }
    }
    ```

2. Invalide el proveedor de serialización predeterminada con `JsonSerializationProvider` en el caso de un agente de escucha de comunicación remota.

   ```csharp
   protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
   {
       return new[]
       {
           new ServiceInstanceListener((c) =>
           {
               return new FabricTransportServiceRemotingListener(c, this,
                   new ServiceRemotingJsonSerializationProvider());
           })
       };
   }
   ```

3. Invalide el proveedor de serialización predeterminada con `JsonSerializationProvider` en el caso de una fábrica de cliente de comunicación remota.

    ```csharp
    var proxyFactory = new ServiceProxyFactory((c) =>
    {
        return new FabricTransportServiceRemotingClientFactory(
        serializationProvider: new ServiceRemotingJsonSerializationProvider());
      });
      ```

## <a name="next-steps"></a>Pasos siguientes

* [Web API con OWIN en Reliable Services](./service-fabric-reliable-services-communication-aspnetcore.md)
* [Comunicación de Windows Communication Foundation con Reliable Services](service-fabric-reliable-services-communication-wcf.md)
* [Protección de la comunicación para Reliable Services](service-fabric-reliable-services-secure-communication.md)
