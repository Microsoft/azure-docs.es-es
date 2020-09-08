---
title: Generación de eventos de registro desde una aplicación .NET
description: Obtenga información acerca de cómo agregar el registro a la aplicación .NET de Service Fabric hospedada en un clúster de Azure o en un clúster independiente.
author: srrengar
ms.topic: conceptual
ms.date: 03/27/2018
ms.author: srrengar
ms.custom: devx-track-csharp
ms.openlocfilehash: a36425acf42a469c7f48b2e954bdacfdfcce1b10
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89011487"
---
# <a name="add-logging-to-your-service-fabric-application"></a>Adición del registro a la aplicación de Service Fabric

La aplicación debe proporcionar suficiente información para depurarla cuando surjan problemas. El registro es uno de los elementos más importantes que se puede agregar a la aplicación de Service Fabric. Cuando se produce un error, un buen registro puede ser una manera de investigar errores. El análisis de patrones de registro permite buscar maneras de mejorar el rendimiento o el diseño de la aplicación. Este documento muestra algunas opciones de registro diferentes.

## <a name="eventflow"></a>EventFlow

El conjunto de la [biblioteca EventFlow](https://github.com/Azure/diagnostics-eventflow) permite a las aplicaciones determinar qué datos de diagnóstico recopilar y dónde se deben enviar. Los datos de diagnóstico pueden ser de cualquier tipo, desde contadores de rendimiento hasta seguimientos de la aplicación. Se ejecuta en el mismo proceso que la aplicación, por lo que se minimiza la sobrecarga de la comunicación. Para obtener más información acerca de EventFlow y Service Fabric, consulte el tema sobre [la adición de eventos de Azure Service Fabric EventFlow](service-fabric-diagnostics-event-aggregation-eventflow.md).

### <a name="using-structured-eventsource-events"></a>Uso de eventos EventSource estructurados

La definición de eventos de mensajes por caso de uso permite recopilar datos sobre el evento en el contexto del evento. Puede buscar y filtrar datos más fácilmente en función de los nombres o los valores de las propiedades de evento especificadas. La estructuración del resultado de la instrumentación facilita la lectura, pero se requiere más tiempo y reflexión para definir un evento para cada caso de uso. 

Algunas definiciones de eventos se pueden compartir en toda la aplicación. Por ejemplo, el evento de inicio o detención de un método se podría reutilizar en muchos servicios de una aplicación. Un servicio específico de dominio, como un sistema de pedidos, puede tener un evento **CreateOrder** con su propio evento único. Este enfoque genera a menudo una gran cantidad de eventos y puede necesitar la coordinación de los identificadores entre los equipos de proyecto. 

```csharp
[EventSource(Name = "MyCompany-VotingState-VotingStateService")]
internal sealed class ServiceEventSource : EventSource
{
    public static readonly ServiceEventSource Current = new ServiceEventSource();

    // The instance constructor is private to enforce singleton semantics.
    private ServiceEventSource() : base() { }

    ...

    // The ServiceTypeRegistered event contains a unique identifier, an event attribute that defined the event, and the code implementation of the event.
    private const int ServiceTypeRegisteredEventId = 3;
    [Event(ServiceTypeRegisteredEventId, Level = EventLevel.Informational, Message = "Service host process {0} registered service type {1}", Keywords = Keywords.ServiceInitialization)]
    public void ServiceTypeRegistered(int hostProcessId, string serviceType)
    {
        WriteEvent(ServiceTypeRegisteredEventId, hostProcessId, serviceType);
    }

    // The ServiceHostInitializationFailed event contains a unique identifier, an event attribute that defined the event, and the code implementation of the event.
    private const int ServiceHostInitializationFailedEventId = 4;
    [Event(ServiceHostInitializationFailedEventId, Level = EventLevel.Error, Message = "Service host initialization failed", Keywords = Keywords.ServiceInitialization)]
    public void ServiceHostInitializationFailed(string exception)
    {
        WriteEvent(ServiceHostInitializationFailedEventId, exception);
    }

    ...

```

### <a name="using-eventsource-generically"></a>Uso de EventSource genéricamente

Dado que la definición de eventos específicos puede resultar difícil, muchas personas definen algunos con un conjunto común de parámetros que por lo general generan la información saliente como cadena. Gran parte de la estructura se pierde, lo que dificulta la búsqueda y el filtrado de los resultados. Con este enfoque se definen algunos eventos que normalmente corresponden a los niveles de registro. El fragmento de código siguiente define un mensaje de error y de depuración:

```csharp
[EventSource(Name = "MyCompany-VotingState-VotingStateService")]
internal sealed class ServiceEventSource : EventSource
{
    public static readonly ServiceEventSource Current = new ServiceEventSource();

    // The Instance constructor is private, to enforce singleton semantics.
    private ServiceEventSource() : base() { }

    ...

    private const int DebugEventId = 10;
    [Event(DebugEventId, Level = EventLevel.Verbose, Message = "{0}")]
    public void Debug(string msg)
    {
        WriteEvent(DebugEventId, msg);
    }

    private const int ErrorEventId = 11;
    [Event(ErrorEventId, Level = EventLevel.Error, Message = "Error: {0} - {1}")]
    public void Error(string error, string msg)
    {
        WriteEvent(ErrorEventId, error, msg);
    }

    ...

```

El uso de instrumentación híbrida (estructurada y genérica) también puede funcionar. La instrumentación estructurada se usa para informar sobre errores y métricas. Los eventos genéricos pueden incluirse en el registro detallado para la solución de problemas por parte de los ingenieros.

## <a name="microsoftextensionslogging"></a>Microsoft.Extensions.Logging

El registro de ASP.NET Core ([paquete NuGet Microsoft.Extensions.Logging](https://www.nuget.org/packages/Microsoft.Extensions.Logging)) es una plataforma de registro que proporciona una API de registro estándar para la aplicación. Es posible conectar el soporte técnico de otros back-ends de registro al registro de ASP.NET Core. Esto ofrece una amplia variedad de soporte técnico para el registro de la aplicación, sin tener que cambiar una gran cantidad de código.

1. Agregue el paquete NuGet **Microsoft.Extensions.Logging** al proyecto que quiera instrumentar. Asimismo, agregue los paquetes de proveedor. Consulte el artículo sobre el [registro de ASP.NET Core](/aspnet/core/fundamentals/logging) para más información.
2. Agregue una directiva **using** para **Microsoft.Extensions.Logging** al archivo de servicio.
3. Defina una variable privada dentro de la clase de servicio.

   ```csharp
   private ILogger _logger = null;
   ```

4. En el constructor de la clase de servicio, agregue este código:

   ```csharp
   _logger = new LoggerFactory().CreateLogger<Stateless>();
   ```

5. Inicie la instrumentación del código en los métodos. Estos son algunos ejemplos:

   ```csharp
   _logger.LogDebug("Debug-level event from Microsoft.Logging");
   _logger.LogInformation("Informational-level event from Microsoft.Logging");

   // In this variant, we're adding structured properties RequestName and Duration, which have values MyRequest and the duration of the request.
   // Later in the article, we discuss why this step is useful.
   _logger.LogInformation("{RequestName} {Duration}", "MyRequest", requestDuration);
   ```

### <a name="using-other-logging-providers"></a>Uso de otros proveedores de registro

Algunos proveedores de terceros usan el enfoque descrito en la sección anterior, como [Serilog](https://serilog.net/), [NLog](https://nlog-project.org/) y [Loggr](https://github.com/imobile3/Loggr.Extensions.Logging). Puede conectarlos al registro de ASP.NET Core o utilizarlos por separado. Serilog tiene una característica que enriquece todos los mensajes enviados desde un registrador. Esta característica puede ser útil para generar el nombre del servicio, el tipo y la información de la partición. Para utilizar esta funcionalidad en la infraestructura de ASP.NET Core, siga estos pasos:

1. Agregue los paquetes NuGet **Serilog**, **Serilog.Extensions.Logging**, **Serilog.Sinks.Literate** y **Serilog.Sinks.Observable** al proyecto. 
2. Cree un argumento `LoggerConfiguration` y la instancia del registrador.

   ```csharp
   Log.Logger = new LoggerConfiguration().WriteTo.LiterateConsole().CreateLogger();
   ```

3. Agregue un argumento `Serilog.ILogger` al constructor de servicio y pase el registrador recién creado.

   ```csharp
   ServiceRuntime.RegisterServiceAsync("StatelessType", context => new Stateless(context, Log.Logger)).GetAwaiter().GetResult();
   ```

4. En el constructor de servicio, cree enriquecedores de propiedades para las propiedades **ServiceTypeName**, **ServiceName**, **PartitionId** e **InstanceId**.

   ```csharp
   public Stateless(StatelessServiceContext context, Serilog.ILogger serilog)
       : base(context)
   {
       PropertyEnricher[] properties = new PropertyEnricher[]
       {
           new PropertyEnricher("ServiceTypeName", context.ServiceTypeName),
           new PropertyEnricher("ServiceName", context.ServiceName),
           new PropertyEnricher("PartitionId", context.PartitionId),
           new PropertyEnricher("InstanceId", context.ReplicaOrInstanceId),
       };

       serilog.ForContext(properties);

       _logger = new LoggerFactory().AddSerilog(serilog.ForContext(properties)).CreateLogger<Stateless>();
   }
   ```

5. Instrumente el código como su estuviera usando ASP.NET Core sin Serilog.

   >[!NOTE]
   >*No* es recomendable usar el argumento `Log.Logger` estático con el ejemplo anterior. Service Fabric puede hospedar varias instancias del mismo tipo de servicio dentro de un único proceso. Si usa el argumento `Log.Logger` estático, el último escritor de enriquecedores de propiedades mostrará los valores de todas las instancias en ejecución. Este es uno de los motivos por los cuales la variable _logger es una variable de miembro privada de la clase de servicio. Además, debe poner `_logger` a disposición del código común, que podría utilizarse en diferentes servicios.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información acerca de la [supervisión de aplicaciones en Service Fabric](service-fabric-diagnostics-event-generation-app.md).
- Obtenga información acerca del registro con [EventFlow](service-fabric-diagnostics-event-aggregation-eventflow.md) y [Microsoft Azure Diagnostics](service-fabric-diagnostics-event-aggregation-wad.md).
