---
title: Introducción a la comunicación de Reliable Services
description: Información general sobre el modelo de comunicación de Reliable Services, incluidos los agentes de escucha de apertura en los servicios, la resolución de puntos de conexión y la comunicación entre servicios.
author: vturecek
ms.topic: conceptual
ms.date: 11/01/2017
ms.author: vturecek
ms.custom: devx-track-csharp
ms.openlocfilehash: 9b45ceaed9f0d3d64a0fc6890549542acc6b1c21
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89018644"
---
# <a name="how-to-use-the-reliable-services-communication-apis"></a>Uso de las API de comunicación de Reliable Services
Azure Service Fabric como una plataforma es completamente independiente de la comunicación entre los servicios. Todos los protocolos y las pilas son aceptables, desde UDP hasta HTTP. El desarrollador del servicio es quien debe elegir cómo deberían comunicarse los servicios. El marco de trabajo de aplicaciones de Reliable Services ofrece pilas de comunicación integradas, además de varias API que puede usar para compilar los componentes de comunicación personalizados.

## <a name="set-up-service-communication"></a>Configuración de la comunicación del servicio
La API de Reliable Services usa una interfaz simple para la comunicación del servicio. Para abrir un punto de conexión para el servicio, solo tiene que implementar esta interfaz:

```csharp

public interface ICommunicationListener
{
    Task<string> OpenAsync(CancellationToken cancellationToken);

    Task CloseAsync(CancellationToken cancellationToken);

    void Abort();
}

```

```java
public interface CommunicationListener {
    CompletableFuture<String> openAsync(CancellationToken cancellationToken);

    CompletableFuture<?> closeAsync(CancellationToken cancellationToken);

    void abort();
}
```

Luego puede agregar su implementación del agente de escucha de comunicación devolviéndolo en una invalidación del método de clase basado en el servicio.

Para servicios sin estado:

```csharp
public class MyStatelessService : StatelessService
{
    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        ...
    }
    ...
}
```
```java
public class MyStatelessService extends StatelessService {

    @Override
    protected List<ServiceInstanceListener> createServiceInstanceListeners() {
        ...
    }
    ...
}
```

Para servicios con estado:

```java
    @Override
    protected List<ServiceReplicaListener> createServiceReplicaListeners() {
        ...
    }
    ...
```

```csharp
public class MyStatefulService : StatefulService
{
    protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
    {
        ...
    }
    ...
}
```

En ambos casos, devuelve una colección de agentes de escucha. Esto permite que el servicio escuche en varios puntos de conexión, que posiblemente usen distintos protocolos, mediante el uso de varios agentes de escucha. Por ejemplo, puede tener un agente de escucha HTTP y un agente de escucha de WebSocket independiente. Cada agente de escucha recibe un nombre y la colección resultante de los pares *nombre : dirección* se representan como un objeto JSON cuando un cliente solicita las direcciones de escucha para una partición o instancia de servicio.

En un servicio sin estado, la invalidación devuelve una colección de ServiceInstanceListeners. Un elemento `ServiceInstanceListener` contiene una función para crear un `ICommunicationListener(C#) / CommunicationListener(Java)` y le asigna un nombre. Para servicios con estado, la invalidación devuelve una colección de ServiceReplicaListeners. Difiere ligeramente de su homólogo sin estado, porque elemento `ServiceReplicaListener` tiene una opción de abrir un elemento `ICommunicationListener` en las réplicas secundarias. No solo puede usar varios agentes de escucha de comunicación en un servicio, sino también especificar cuáles aceptan solicitudes en réplicas secundarias y cuáles escuchan solo en las réplicas principales.

Por ejemplo, puede tener una clase ServiceRemotingListener que realice llamadas RPC solo en las réplicas principales, y un segundo agente de escucha personalizado que lea solicitudes en réplicas secundarias a través de HTTP:

```csharp
protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
{
    return new[]
    {
        new ServiceReplicaListener(context =>
            new MyCustomHttpListener(context),
            "HTTPReadonlyEndpoint",
            true),

        new ServiceReplicaListener(context =>
            this.CreateServiceRemotingListener(context),
            "rpcPrimaryEndpoint",
            false)
    };
}
```

> [!NOTE]
> Cuando se crean varios agentes de escucha para un servicio, se **debe** asignar a cada uno un nombre único.
>
>

Por último, describa los puntos de conexión que se requieren para el servicio en el [manifiesto de servicio](service-fabric-application-and-service-manifests.md) en la sección que trata sobre los puntos de conexión.

```xml
<Resources>
    <Endpoints>
      <Endpoint Name="WebServiceEndpoint" Protocol="http" Port="80" />
      <Endpoint Name="OtherServiceEndpoint" Protocol="tcp" Port="8505" />
    <Endpoints>
</Resources>

```

El agente de escucha de comunicación puede acceder a los recursos de puntos de conexión asignados a él desde `CodePackageActivationContext` en `ServiceContext`. El agente de escucha luego puede empezar a escuchar las solicitudes cuando se abre.

```csharp
var codePackageActivationContext = serviceContext.CodePackageActivationContext;
var port = codePackageActivationContext.GetEndpoint("ServiceEndpoint").Port;

```
```java
CodePackageActivationContext codePackageActivationContext = serviceContext.getCodePackageActivationContext();
int port = codePackageActivationContext.getEndpoint("ServiceEndpoint").getPort();

```

> [!NOTE]
> Los recursos de los puntos de conexión son comunes para el paquete de servicio completo y los asigna Service Fabric cuando se activa el paquete de servicio. Es posible que varias réplicas hospedadas en el mismo ServiceHost compartan el mismo puerto. Esto significa que el agente de escucha de comunicación debe admitir el uso compartido de puertos. La manera recomendada de hacerlo es que el agente de escucha de comunicación utilice el identificador de partición y el identificador de instancia o de réplica cuando genera la dirección de escucha.
>
>

### <a name="service-address-registration"></a>Registro de dirección de servicio
Un servicio del sistema denominado *servicio de nomenclatura* se ejecuta en clústeres de Service Fabric. El servicio de nomenclatura es un registrador de los servicios y sus direcciones que cada instancia o réplica del servicio escucha. Cuando el método `OpenAsync(C#) / openAsync(Java)` de una interfaz `ICommunicationListener(C#) / CommunicationListener(Java)` se completa, el valor devuelto se registra en el servicio de nomenclatura. Este valor devuelto que se publica en el servicio de nomenclatura es una cadena que puede no tener ningún valor. Este valor de cadena es lo que ven los clientes cuando solicitan una dirección para el servicio desde el servicio de nomenclatura.

```csharp
public Task<string> OpenAsync(CancellationToken cancellationToken)
{
    EndpointResourceDescription serviceEndpoint = serviceContext.CodePackageActivationContext.GetEndpoint("ServiceEndpoint");
    int port = serviceEndpoint.Port;

    this.listeningAddress = string.Format(
                CultureInfo.InvariantCulture,
                "http://+:{0}/",
                port);

    this.publishAddress = this.listeningAddress.Replace("+", FabricRuntime.GetNodeContext().IPAddressOrFQDN);

    this.webApp = WebApp.Start(this.listeningAddress, appBuilder => this.startup.Invoke(appBuilder));

    // the string returned here will be published in the Naming Service.
    return Task.FromResult(this.publishAddress);
}
```
```java
public CompletableFuture<String> openAsync(CancellationToken cancellationToken)
{
    EndpointResourceDescription serviceEndpoint = serviceContext.getCodePackageActivationContext.getEndpoint("ServiceEndpoint");
    int port = serviceEndpoint.getPort();

    this.publishAddress = String.format("http://%s:%d/", FabricRuntime.getNodeContext().getIpAddressOrFQDN(), port);

    this.webApp = new WebApp(port);
    this.webApp.start();

    /* the string returned here will be published in the Naming Service.
     */
    return CompletableFuture.completedFuture(this.publishAddress);
}
```

Service Fabric ofrece varias API que permiten que los clientes y otros servicios soliciten esta dirección por nombre de servicio. Esto es importante porque la dirección del servicio no es estática. Los servicios se mueven en el clúster para fines de disponibilidad y equilibrio de recursos. Este mecanismo permite a los clientes resolver la dirección de escucha de un servicio.

> [!NOTE]
> Para ver un tutorial completo sobre cómo escribir un agente de escucha de comunicación, consulte el artículo sobre los [servicios de la API web de Service Fabric con autohospedaje OWIN](./service-fabric-reliable-services-communication-aspnetcore.md) para C#, mientras que para Java puede escribir su propia implementación del servidor HTTP; consulte el ejemplo de aplicación de EchoServer en https://github.com/Azure-Samples/service-fabric-java-getting-started.
>
>

## <a name="communicating-with-a-service"></a>Comunicación con un servicio
La API de Reliable Services proporciona las bibliotecas siguientes para la escritura de clientes que se comunican con los servicios.

### <a name="service-endpoint-resolution"></a>Resolución del punto de conexión de servicio
El primer paso para la comunicación con un servicio consiste en resolver una dirección de punto de conexión de la partición o la instancia del servicio con el que quiere comunicarse. La clase de utilidad `ServicePartitionResolver(C#) / FabricServicePartitionResolver(Java)` es un tipo primitivo básico que ayuda a los clientes a determinar el punto de conexión de un servicio en tiempo de ejecución. En la terminología de Service Fabric, el proceso de determinar el punto de conexión de un servicio se denomina *resolución de punto de conexión de servicio*.

Para conectarse a servicios en un mismo clúster, se puede crear ServicePartitionResolver con la configuración predeterminada. Este es el uso recomendado para la mayoría de las situaciones:

```csharp
ServicePartitionResolver resolver = ServicePartitionResolver.GetDefault();
```
```java
FabricServicePartitionResolver resolver = FabricServicePartitionResolver.getDefault();
```

Para conectarse a servicios en otro clúster, se puede crear ServicePartitionResolver con un conjunto de puntos de conexión de puerta de enlace del clúster. Tenga en cuenta que los puntos de conexión de puerta de enlace son solo distintos puntos de conexión para conectarse al mismo clúster. Por ejemplo:

```csharp
ServicePartitionResolver resolver = new  ServicePartitionResolver("mycluster.cloudapp.azure.com:19000", "mycluster.cloudapp.azure.com:19001");
```
```java
FabricServicePartitionResolver resolver = new  FabricServicePartitionResolver("mycluster.cloudapp.azure.com:19000", "mycluster.cloudapp.azure.com:19001");
```

Como alternativa, puede asignarse una función a `ServicePartitionResolver` para crear un objeto `FabricClient` para uso interno:

```csharp
public delegate FabricClient CreateFabricClientDelegate();
```
```java
public FabricServicePartitionResolver(CreateFabricClient createFabricClient) {
...
}

public interface CreateFabricClient {
    public FabricClient getFabricClient();
}
```

`FabricClient` es el objeto que se utiliza para comunicarse con el clúster de Service Fabric para realizar diversas operaciones de administración en el clúster. Esto resulta útil cuando se desea tener más control sobre cómo interactúa la resolución de la partición del servicio con el clúster. `FabricClient` realiza el almacenamiento en caché internamente y, por lo general, su creación es más costosa, por lo que es importante reutilizar instancias de `FabricClient` tantas veces como sea posible.

```csharp
ServicePartitionResolver resolver = new  ServicePartitionResolver(() => CreateMyFabricClient());
```
```java
FabricServicePartitionResolver resolver = new  FabricServicePartitionResolver(() -> new CreateFabricClientImpl());
```

Luego se usa un método de resolución para recuperar la dirección de un servicio o una partición de servicio en el caso de los servicios con particiones.

```csharp
ServicePartitionResolver resolver = ServicePartitionResolver.GetDefault();

ResolvedServicePartition partition =
    await resolver.ResolveAsync(new Uri("fabric:/MyApp/MyService"), new ServicePartitionKey(), cancellationToken);
```
```java
FabricServicePartitionResolver resolver = FabricServicePartitionResolver.getDefault();

CompletableFuture<ResolvedServicePartition> partition =
    resolver.resolveAsync(new URI("fabric:/MyApp/MyService"), new ServicePartitionKey());
```

Una dirección de servicio se puede resolver fácilmente con una clase ServicePartitionResolver, pero se requiere más trabajo para garantizar que la dirección resuelta puede usarse correctamente. El cliente tiene que detectar si el intento de conexión no se realizó debido a un error transitorio y se puede volver a intentar (por ejemplo, el servicio se movió o no está disponible temporalmente) o se debe a un error permanente (por ejemplo, el servicio se eliminó o el recurso solicitado ya no existe). Las instancias o réplicas de servicio pueden moverse siempre de un nodo a otro por varios motivos. La dirección de servicio que se resuelve a través de ServicePartitionResolver puede estar obsoleta en el momento en que el código de cliente intenta conectarse. En ese caso, el cliente tiene que volver a resolver la dirección. Al proporcionar la clase `ResolvedServicePartition` anterior, se indica que es necesario volver a intentar la resolución en lugar de simplemente recuperar una dirección en caché.

Normalmente, el código de cliente no necesita trabajar directamente con ServicePartitionResolver. Se crea y se pasa a otras fábricas de cliente de comunicación en la API de Reliable Services. Las fábricas usan la resolución internamente para generar un objeto de cliente que puede utilizarse para comunicarse con servicios.

### <a name="communication-clients-and-factories"></a>Fábricas y clientes de comunicación
La biblioteca de fábrica de comunicación implementa un patrón de reintento típico de control de errores que simplifica el reintento de conexiones en puntos de conexión de servicio resueltos. La biblioteca de fábrica proporciona el mecanismo de reintento, mientras que usted proporciona los controladores de errores.

`ICommunicationClientFactory(C#) / CommunicationClientFactory(Java)` define la interfaz base que implementa una fábrica de cliente de comunicación que genera clientes que pueden comunicarse con un servicio de Service Fabric. La implementación de CommunicationClientFactory depende de la pila de comunicación que el servicio de Service Fabric utiliza donde el cliente desea comunicarse. La API de Reliable Services ofrece una instancia de `CommunicationClientFactoryBase<TCommunicationClient>`. Esto proporciona una implementación base de la interfaz CommunicationClientFactory y realiza tareas comunes a todas las pilas de comunicación. (Estas tareas incluyen el uso de un ServicePartitionResolver para determinar el punto de conexión de servicio). Normalmente, los clientes implementan la clase CommunicationClientFactoryBase abstracta para controlar la lógica específica de la pila de comunicación.

El cliente de comunicación solo recibe una dirección y la usa para conectarse a un servicio. El cliente puede utilizar el protocolo que quiera.

```csharp
public class MyCommunicationClient : ICommunicationClient
{
    public ResolvedServiceEndpoint Endpoint { get; set; }

    public string ListenerName { get; set; }

    public ResolvedServicePartition ResolvedServicePartition { get; set; }
}
```
```java
public class MyCommunicationClient implements CommunicationClient {

    private ResolvedServicePartition resolvedServicePartition;
    private String listenerName;
    private ResolvedServiceEndpoint endPoint;

    /*
     * Getters and Setters
     */
}
```

La fábrica de cliente es la principal responsable de crear clientes de comunicación. En el caso de los clientes que no mantienen una conexión persistente, como un cliente HTTP, la fábrica solo tiene que crear y devolver el cliente. La fábrica también debe validar otros protocolos que mantienen una conexión persistente, como algunos de los protocolos binarios, para determinar si la conexión debe volver a crearse.  

```csharp
public class MyCommunicationClientFactory : CommunicationClientFactoryBase<MyCommunicationClient>
{
    protected override void AbortClient(MyCommunicationClient client)
    {
    }

    protected override Task<MyCommunicationClient> CreateClientAsync(string endpoint, CancellationToken cancellationToken)
    {
    }

    protected override bool ValidateClient(MyCommunicationClient clientChannel)
    {
    }

    protected override bool ValidateClient(string endpoint, MyCommunicationClient client)
    {
    }
}
```
```java
public class MyCommunicationClientFactory extends CommunicationClientFactoryBase<MyCommunicationClient> {

    @Override
    protected boolean validateClient(MyCommunicationClient clientChannel) {
    }

    @Override
    protected boolean validateClient(String endpoint, MyCommunicationClient client) {
    }

    @Override
    protected CompletableFuture<MyCommunicationClient> createClientAsync(String endpoint) {
    }

    @Override
    protected void abortClient(MyCommunicationClient client) {
    }
}
```

Por último, un controlador de excepciones es responsable de determinar la acción que hay que realizar cuando se produce una excepción. Las excepciones se dividen en dos categorías, las que **se pueden volver a intentar** y las que **no se pueden volver a intentar**.

* Las que **se pueden volver a intentar** simplemente se reenvían al autor de la llamada.
* Las excepciones que **se pueden volver a intentar** se clasifican a su vez en **transitorias** y **no transitorias**.
  * **transitorias** son las que pueden reintentarse sin volver a resolver la dirección del punto de conexión de servicio. Entre estas últimas se incluyen los problemas transitorios de red o las respuestas de error de servicio que no sean las que indican que la dirección de punto de conexión de servicio no existe.
  * **Non-transient** son las que requieren que se vuelva a resolver la dirección de punto de conexión de servicio. Entre estas se incluyen las excepciones que indican que no se pudo llegar al punto de conexión de servicio, lo que indica que el servicio se movió a otro nodo.

El método `TryHandleException` toma una decisión sobre una excepción determinada. Si **no sabe** qué decisiones tomar sobre una excepción, debe devolver **false**. Cuando **sí sabe** qué decisión tomar, debe establecer el resultado según corresponda y devolver **true**.

```csharp
class MyExceptionHandler : IExceptionHandler
{
    public bool TryHandleException(ExceptionInformation exceptionInformation, OperationRetrySettings retrySettings, out ExceptionHandlingResult result)
    {
        // if exceptionInformation.Exception is known and is transient (can be retried without re-resolving)
        result = new ExceptionHandlingRetryResult(exceptionInformation.Exception, true, retrySettings, retrySettings.DefaultMaxRetryCount);
        return true;


        // if exceptionInformation.Exception is known and is not transient (indicates a new service endpoint address must be resolved)
        result = new ExceptionHandlingRetryResult(exceptionInformation.Exception, false, retrySettings, retrySettings.DefaultMaxRetryCount);
        return true;

        // if exceptionInformation.Exception is unknown (let the next IExceptionHandler attempt to handle it)
        result = null;
        return false;
    }
}
```
```java
public class MyExceptionHandler implements ExceptionHandler {

    @Override
    public ExceptionHandlingResult handleException(ExceptionInformation exceptionInformation, OperationRetrySettings retrySettings) {

        /* if exceptionInformation.getException() is known and is transient (can be retried without re-resolving)
         */
        result = new ExceptionHandlingRetryResult(exceptionInformation.getException(), true, retrySettings, retrySettings.getDefaultMaxRetryCount());
        return true;


        /* if exceptionInformation.getException() is known and is not transient (indicates a new service endpoint address must be resolved)
         */
        result = new ExceptionHandlingRetryResult(exceptionInformation.getException(), false, retrySettings, retrySettings.getDefaultMaxRetryCount());
        return true;

        /* if exceptionInformation.getException() is unknown (let the next ExceptionHandler attempt to handle it)
         */
        result = null;
        return false;

    }
}
```
### <a name="putting-it-all-together"></a>Resumen
Con `ICommunicationClient(C#) / CommunicationClient(Java)`, `ICommunicationClientFactory(C#) / CommunicationClientFactory(Java)` e `IExceptionHandler(C#) / ExceptionHandler(Java)` creadas en torno a un protocolo de comunicaciones, `ServicePartitionClient(C#) / FabricServicePartitionClient(Java)` lo encapsula todo y proporciona el bucle de resolución de direcciones de partición de servicio y control de errores en torno a estos componentes.

```csharp
private MyCommunicationClientFactory myCommunicationClientFactory;
private Uri myServiceUri;

var myServicePartitionClient = new ServicePartitionClient<MyCommunicationClient>(
    this.myCommunicationClientFactory,
    this.myServiceUri,
    myPartitionKey);

var result = await myServicePartitionClient.InvokeWithRetryAsync(async (client) =>
   {
      // Communicate with the service using the client.
   },
   CancellationToken.None);

```
```java
private MyCommunicationClientFactory myCommunicationClientFactory;
private URI myServiceUri;

FabricServicePartitionClient myServicePartitionClient = new FabricServicePartitionClient<MyCommunicationClient>(
    this.myCommunicationClientFactory,
    this.myServiceUri,
    myPartitionKey);

CompletableFuture<?> result = myServicePartitionClient.invokeWithRetryAsync(client -> {
      /* Communicate with the service using the client.
       */
   });

```

## <a name="next-steps"></a>Pasos siguientes
* [ASP.NET Core con Reliable Services](service-fabric-reliable-services-communication-aspnetcore.md)
* [Llamadas a procedimiento remoto con la comunicación remota de Reliable Services](service-fabric-reliable-services-communication-remoting.md)
* [Comunicación WCF con la utilización de Reliable Services](service-fabric-reliable-services-communication-wcf.md)
