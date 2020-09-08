---
title: Seguimiento de las operaciones personalizadas con el SDK de .NET de Azure Application Insights
description: Seguimiento de las operaciones personalizadas con el SDK de .NET de Azure Application Insights
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 11/26/2019
ms.reviewer: sergkanz
ms.openlocfilehash: 42a5318325f9961483465357403089755feb130d
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88933314"
---
# <a name="track-custom-operations-with-application-insights-net-sdk"></a>Seguimiento de las operaciones personalizadas con el SDK de .NET para Application Insights

Los SDK de Azure Application Insights realizan el seguimiento automático de las solicitudes HTTP de entrada y las llamadas a los servicios dependientes, como solicitudes HTTP y consultas SQL. El seguimiento y la correlación de las solicitudes y dependencias le ofrece visibilidad en la capacidad de respuesta de toda la aplicación y la confiabilidad en todos los microservicios que combinan esta aplicación. 

Hay una clase de patrones de aplicación que no se admiten de forma genérica. La supervisión correcta de estos patrones requiere la instrumentación manual de código. En este artículo se abordan algunos patrones que pueden requerir la instrumentación manual, como el procesamiento de colas personalizadas y la ejecución de tareas de larga duración en segundo plano.

En este documento se proporcionan instrucciones sobre cómo realizar el seguimiento de operaciones personalizadas con el SDK de Application Insights. Esta documentación es relevante para:

- Application Insights para .NET (también conocido como Base SDK) versión 2.4+.
- Application Insights para aplicaciones web (con ASP.NET) versión 2.4+.
- Application Insights para ASP.NET Core versión 2.1+.

## <a name="overview"></a>Información general
Una operación es una parte de trabajo lógica ejecutada por una aplicación. Tiene un nombre, una hora de inicio, una duración, un resultado y un contexto de ejecución como el nombre de usuario, las propiedades y el resultado. Si la operación B inició la operación A, la operación B se establece como un elemento primario de A. Una operación solo puede tener un elemento primario, pero puede tener muchas operaciones secundarias. Para más información sobre las operaciones y la correlación de telemetría, vea [Correlación de telemetría en Azure Application Insights](correlation.md).

En el SDK de .NET para Application Insights se describe la operación mediante la clase abstracta [OperationTelemetry](https://github.com/microsoft/ApplicationInsights-dotnet/blob/7633ae849edc826a8547745b6bf9f3174715d4bd/BASE/src/Microsoft.ApplicationInsights/Extensibility/Implementation/OperationTelemetry.cs) y sus descendientes [RequestTelemetry](https://github.com/microsoft/ApplicationInsights-dotnet/blob/7633ae849edc826a8547745b6bf9f3174715d4bd/BASE/src/Microsoft.ApplicationInsights/DataContracts/RequestTelemetry.cs) y [DependencyTelemetry](https://github.com/microsoft/ApplicationInsights-dotnet/blob/7633ae849edc826a8547745b6bf9f3174715d4bd/BASE/src/Microsoft.ApplicationInsights/DataContracts/DependencyTelemetry.cs).

## <a name="incoming-operations-tracking"></a>Seguimiento de operaciones de entrada 
El SDK web de Application Insights recopila automáticamente las solicitudes HTTP para las aplicaciones ASP.NET que se ejecutan en una canalización IIS y todas las aplicaciones ASP.NET Core. Hay soluciones admitidas por la comunidad para otras plataformas y marcos de trabajo. Pero si la aplicación no es compatible con ninguna solución admitida por los estándares o la comunidad, puede instrumentarla de forma manual.

Otro ejemplo que requiere seguimiento personalizado es el proceso de trabajo que recibe los elementos de la cola. Para algunas colas, se realiza un seguimiento como dependencia de la llamada para agregar un mensaje a esta cola. Pero la operación general que describe el procesamiento de mensajes no se recopila automáticamente.

Ahora vamos a ver cómo se puede realizar un seguimiento de tales operaciones.

Generalmente, la tarea consiste en crear `RequestTelemetry` y establecer propiedades conocidas. Una vez finalizada la operación, se realiza el seguimiento de la telemetría. En el siguiente ejemplo se muestra esta tarea.

### <a name="http-request-in-owin-self-hosted-app"></a>Solicitud HTTP en una aplicación autohospedada de Owin
En este ejemplo, el contexto de seguimiento se propaga según el [protocolo HTTP para la correlación](https://github.com/dotnet/runtime/blob/master/src/libraries/System.Diagnostics.DiagnosticSource/src/HttpCorrelationProtocol.md). Debe esperar recibir los encabezados que se describen ahí.

```csharp
public class ApplicationInsightsMiddleware : OwinMiddleware
{
    // you may create a new TelemetryConfiguration instance, reuse one you already have
    // or fetch the instance created by Application Insights SDK.
    private readonly TelemetryConfiguration telemetryConfiguration = TelemetryConfiguration.CreateDefault();
    private readonly TelemetryClient telemetryClient = new TelemetryClient(telemetryConfiguration);
    
    public ApplicationInsightsMiddleware(OwinMiddleware next) : base(next) {}

    public override async Task Invoke(IOwinContext context)
    {
        // Let's create and start RequestTelemetry.
        var requestTelemetry = new RequestTelemetry
        {
            Name = $"{context.Request.Method} {context.Request.Uri.GetLeftPart(UriPartial.Path)}"
        };

        // If there is a Request-Id received from the upstream service, set the telemetry context accordingly.
        if (context.Request.Headers.ContainsKey("Request-Id"))
        {
            var requestId = context.Request.Headers.Get("Request-Id");
            // Get the operation ID from the Request-Id (if you follow the HTTP Protocol for Correlation).
            requestTelemetry.Context.Operation.Id = GetOperationId(requestId);
            requestTelemetry.Context.Operation.ParentId = requestId;
        }

        // StartOperation is a helper method that allows correlation of 
        // current operations with nested operations/telemetry
        // and initializes start time and duration on telemetry items.
        var operation = telemetryClient.StartOperation(requestTelemetry);

        // Process the request.
        try
        {
            await Next.Invoke(context);
        }
        catch (Exception e)
        {
            requestTelemetry.Success = false;
            telemetryClient.TrackException(e);
            throw;
        }
        finally
        {
            // Update status code and success as appropriate.
            if (context.Response != null)
            {
                requestTelemetry.ResponseCode = context.Response.StatusCode.ToString();
                requestTelemetry.Success = context.Response.StatusCode >= 200 && context.Response.StatusCode <= 299;
            }
            else
            {
                requestTelemetry.Success = false;
            }

            // Now it's time to stop the operation (and track telemetry).
            telemetryClient.StopOperation(operation);
        }
    }
    
    public static string GetOperationId(string id)
    {
        // Returns the root ID from the '|' to the first '.' if any.
        int rootEnd = id.IndexOf('.');
        if (rootEnd < 0)
            rootEnd = id.Length;

        int rootStart = id[0] == '|' ? 1 : 0;
        return id.Substring(rootStart, rootEnd - rootStart);
    }
}
```

El Protocolo HTTP para la correlación también declara el encabezado `Correlation-Context`. Pero aquí se omite para simplificar.

## <a name="queue-instrumentation"></a>Instrumentación de colas
Aunque existen el [contexto de seguimiento de W3C](https://www.w3.org/TR/trace-context/) y el [protocolo HTTP para la correlación](https://github.com/dotnet/runtime/blob/master/src/libraries/System.Diagnostics.DiagnosticSource/src/HttpCorrelationProtocol.md) para pasar los detalles de la correlación con la solicitud HTTP, todos los protocolos de cola tienen que definir cómo se transmiten los mismos detalles junto con el mensaje de cola. Algunos protocolos de cola (por ejemplo, AMQP) permiten pasar metadatos adicionales, y otros (como una cola de Azure Storage) requieren que el contexto se codifique en la carga del mensaje.

> [!NOTE]
> * **Todavía no se admite el seguimiento entre componentes para las colas** con HTTP, si el productor y el consumidor envían telemetría a distintos recursos de Application Insights, la experiencia de diagnósticos de transacción y el mapa de aplicación muestran transacciones y asignaciones de un extremo a otro. En el caso de las colas, todavía no se admite. 

### <a name="service-bus-queue"></a>Cola de Service Bus
Application Insights realiza un seguimiento de las llamadas de mensajería de Service Bus con la nueva versión 3.0.0 del [cliente Microsoft Azure ServiceBus para .NET](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus/) y con versiones posteriores.
Si usa un [patrón de controlador de mensajes](/dotnet/api/microsoft.azure.servicebus.queueclient.registermessagehandler) para procesar mensajes, ya ha terminado: se realizará el seguimiento automático de todas las llamadas de Service Bus que se hayan hecho mediante el servicio y se correlacionarán con otros elementos de telemetría. Consulte el artículo sobre el [seguimiento del cliente Service Bus con Microsoft Application Insights](../../service-bus-messaging/service-bus-end-to-end-tracing.md) si desea procesar los mensajes manualmente.

Si usa el paquete [WindowsAzure.ServiceBus](https://www.nuget.org/packages/WindowsAzure.ServiceBus/), siga leyendo; en los ejemplos siguientes se muestra cómo realizar un seguimiento de las llamadas y correlacionarlas en Service Bus, ya que la cola de Service Bus usa el protocolo AMQP y Application Insights no realiza un seguimiento automático de las operaciones de cola.
Los identificadores de correlación se transmiten en las propiedades del mensaje.

#### <a name="enqueue"></a>Poner en cola

```csharp
public async Task Enqueue(string payload)
{
    // StartOperation is a helper method that initializes the telemetry item
    // and allows correlation of this operation with its parent and children.
    var operation = telemetryClient.StartOperation<DependencyTelemetry>("enqueue " + queueName);
    
    operation.Telemetry.Type = "Azure Service Bus";
    operation.Telemetry.Data = "Enqueue " + queueName;

    var message = new BrokeredMessage(payload);
    // Service Bus queue allows the property bag to pass along with the message.
    // We will use them to pass our correlation identifiers (and other context)
    // to the consumer.
    message.Properties.Add("ParentId", operation.Telemetry.Id);
    message.Properties.Add("RootId", operation.Telemetry.Context.Operation.Id);

    try
    {
        await queue.SendAsync(message);
        
        // Set operation.Telemetry Success and ResponseCode here.
        operation.Telemetry.Success = true;
    }
    catch (Exception e)
    {
        telemetryClient.TrackException(e);
        // Set operation.Telemetry Success and ResponseCode here.
        operation.Telemetry.Success = false;
        throw;
    }
    finally
    {
        telemetryClient.StopOperation(operation);
    }
}
```

#### <a name="process"></a>Proceso
```csharp
public async Task Process(BrokeredMessage message)
{
    // After the message is taken from the queue, create RequestTelemetry to track its processing.
    // It might also make sense to get the name from the message.
    RequestTelemetry requestTelemetry = new RequestTelemetry { Name = "process " + queueName };

    var rootId = message.Properties["RootId"].ToString();
    var parentId = message.Properties["ParentId"].ToString();
    // Get the operation ID from the Request-Id (if you follow the HTTP Protocol for Correlation).
    requestTelemetry.Context.Operation.Id = rootId;
    requestTelemetry.Context.Operation.ParentId = parentId;

    var operation = telemetryClient.StartOperation(requestTelemetry);

    try
    {
        await ProcessMessage();
    }
    catch (Exception e)
    {
        telemetryClient.TrackException(e);
        throw;
    }
    finally
    {
        // Update status code and success as appropriate.
        telemetryClient.StopOperation(operation);
    }
}
```

### <a name="azure-storage-queue"></a>Cola de Azure Storage
En el ejemplo siguiente se muestra cómo realizar el seguimiento de operaciones de [cola de Azure Storage](../../storage/queues/storage-dotnet-how-to-use-queues.md) y poner en correlación la telemetría entre el productor, el consumidor y Azure Storage. 

La cola de Storage tiene una API HTTP. El recolector de dependencias de Application Insights realiza el seguimiento de todas las llamadas a la cola para las solicitudes HTTP.
Se configura de forma predeterminada en las aplicaciones ASP.NET y ASP.NET Core, con otros tipos de aplicación, puede consultar la [documentación de las aplicaciones de consola](./console.md).

Es posible que también quiera poner en correlación el identificador de operación de Application Insights con el identificador de solicitud de Storage. Para obtener información sobre cómo establecer y obtener un cliente de solicitud de Storage y un identificador de solicitud de servidor, vea [Supervisión, diagnóstico y solución de problemas de Azure Storage](../../storage/common/storage-monitoring-diagnosing-troubleshooting.md#end-to-end-tracing).

#### <a name="enqueue"></a>Poner en cola
Como las colas de Storage admiten la API de HTTP, Application Insights realiza el seguimiento automático de todas las operaciones en la cola. En muchos casos, esta instrumentación debería ser suficiente. Pero para poner en correlación seguimientos en el lado del consumidor con seguimientos del productor, tiene que pasar un contexto de correlación de forma similar a como se hace en el Protocolo HTTP para la correlación. 

En este ejemplo se muestra cómo realizar un seguimiento de la operación `Enqueue`. Puede:

 - **Poner en correlación los reintentos (si existen)** : todos tienen un elemento primario común que es la operación `Enqueue`. En caso contrario, se realiza su seguimiento como elementos secundarios de la solicitud de entrada. Si hay varias solicitudes lógicas a la cola, podría ser difícil buscar qué llamada generó los reintentos.
 - **Poner en correlación los registros de almacenamiento (si es necesario y cuando sea necesario)** : se correlacionan con la telemetría de Application Insights.

La operación `Enqueue` es el elemento secundario de una operación principal (por ejemplo, una solicitud HTTP de entrada). La llamada de dependencia HTTP es el elemento secundario de la operación `Enqueue` y el descendiente de la solicitud de entrada:

```csharp
public async Task Enqueue(CloudQueue queue, string message)
{
    var operation = telemetryClient.StartOperation<DependencyTelemetry>("enqueue " + queue.Name);
    operation.Telemetry.Type = "Azure queue";
    operation.Telemetry.Data = "Enqueue " + queue.Name;

    // MessagePayload represents your custom message and also serializes correlation identifiers into payload.
    // For example, if you choose to pass payload serialized to JSON, it might look like
    // {'RootId' : 'some-id', 'ParentId' : '|some-id.1.2.3.', 'message' : 'your message to process'}
    var jsonPayload = JsonConvert.SerializeObject(new MessagePayload
    {
        RootId = operation.Telemetry.Context.Operation.Id,
        ParentId = operation.Telemetry.Id,
        Payload = message
    });
    
    CloudQueueMessage queueMessage = new CloudQueueMessage(jsonPayload);

    // Add operation.Telemetry.Id to the OperationContext to correlate Storage logs and Application Insights telemetry.
    OperationContext context = new OperationContext { ClientRequestID = operation.Telemetry.Id};

    try
    {
        await queue.AddMessageAsync(queueMessage, null, null, new QueueRequestOptions(), context);
    }
    catch (StorageException e)
    {
        operation.Telemetry.Properties.Add("AzureServiceRequestID", e.RequestInformation.ServiceRequestID);
        operation.Telemetry.Success = false;
        operation.Telemetry.ResultCode = e.RequestInformation.HttpStatusCode.ToString();
        telemetryClient.TrackException(e);
    }
    finally
    {
        // Update status code and success as appropriate.
        telemetryClient.StopOperation(operation);
    }
}  
```

Para reducir la cantidad de telemetría que notifica la aplicación o si no quiere realizar el seguimiento de la operación `Enqueue` por otras razones, use directamente la API `Activity`:

- Cree (e inicie) una nueva `Activity` en lugar de iniciar la operación de Application Insights. *No* es necesario asignar ninguna propiedad en este elemento, excepto el nombre de la operación.
- Serialice `yourActivity.Id` en la carga del mensaje en lugar de `operation.Telemetry.Id`. También se puede usar `Activity.Current.Id`.


#### <a name="dequeue"></a>Quitar de la cola
Al igual que `Enqueue`, Application Insights realiza el seguimiento automático de la solicitud HTTP a la cola de Storage. Pero supuestamente la operación `Enqueue` se produce en el contexto principal, por ejemplo en un contexto de solicitud de entrada. Los SDK de Application Insights correlacionan automáticamente esta operación (y su elemento HTTP) con la solicitud primaria y otra telemetría notificada en el mismo ámbito.

La operación `Dequeue` es complicada. El SDK de Application Insights realiza automáticamente el seguimiento de las solicitudes HTTP. Pero desconoce el contexto de correlación hasta que se analiza el mensaje. No es posible poner en correlación la solicitud HTTP para obtener el mensaje con el resto de la telemetría, en especial cuando se recibe más de un mensaje.

```csharp
public async Task<MessagePayload> Dequeue(CloudQueue queue)
{
    var operation = telemetryClient.StartOperation<DependencyTelemetry>("dequeue " + queue.Name);
    operation.Telemetry.Type = "Azure queue";
    operation.Telemetry.Data = "Dequeue " + queue.Name;
    
    try
    {
        var message = await queue.GetMessageAsync();
    }
    catch (StorageException e)
    {
        operation.telemetry.Properties.Add("AzureServiceRequestID", e.RequestInformation.ServiceRequestID);
        operation.telemetry.Success = false;
        operation.telemetry.ResultCode = e.RequestInformation.HttpStatusCode.ToString();
        telemetryClient.TrackException(e);
    }
    finally
    {
        // Update status code and success as appropriate.
        telemetryClient.StopOperation(operation);
    }

    return null;
}
```

#### <a name="process"></a>Proceso

En el ejemplo siguiente, se realiza el seguimiento de un mensaje de entrada de forma similar a cómo se realiza en una solicitud HTTP de entrada:

```csharp
public async Task Process(MessagePayload message)
{
    // After the message is dequeued from the queue, create RequestTelemetry to track its processing.
    RequestTelemetry requestTelemetry = new RequestTelemetry { Name = "process " + queueName };
    
    // It might also make sense to get the name from the message.
    requestTelemetry.Context.Operation.Id = message.RootId;
    requestTelemetry.Context.Operation.ParentId = message.ParentId;

    var operation = telemetryClient.StartOperation(requestTelemetry);

    try
    {
        await ProcessMessage();
    }
    catch (Exception e)
    {
        telemetryClient.TrackException(e);
        throw;
    }
    finally
    {
        // Update status code and success as appropriate.
        telemetryClient.StopOperation(operation);
    }
}
```

Del mismo modo, se pueden instrumentar otras operaciones de cola. Una operación de lectura se debe instrumentalizar de una manera similar a la de una operación de quitar de la cola. La instrumentación de las operaciones de administración de cola no es necesaria. Application Insights realiza el seguimiento de las operaciones como HTTP y en la mayoría de los casos es suficiente.

Al instrumentar la eliminación de mensajes, asegúrese de establecer los identificadores de la operación (correlación). Como alternativa, puede usar la API de `Activity`. No es necesario establecer identificadores de operaciones en los elementos de telemetría, porque el SDK de Application Insights lo hace automáticamente:

- Cree una nueva `Activity` después de que tenga un elemento de la cola.
- Use `Activity.SetParentId(message.ParentId)` para poner en correlación los registros de consumidor y productor.
- Inicie la `Activity`.
- Realice el seguimiento de las operaciones de quitar de la cola, proceso y eliminación mediante los asistentes `Start/StopOperation`. Hágalo desde el mismo flujo de control asincrónico (contexto de ejecución). De esta forma, se correlacionan correctamente.
- Pare la `Activity`.
- Use `Start/StopOperation` o llame a la telemetría de `Track` manualmente.

### <a name="dependency-types"></a>Tipos de dependencia

Application Insights usa el tipo de dependencia para personalizar las experiencias de la interfaz de usuario. En el caso de las colas, reconoce los siguientes tipos de `DependencyTelemetry` que mejoran la [experiencia de diagnósticos de transacción](./transaction-diagnostics.md):
- `Azure queue` para colas de Azure Storage
- `Azure Event Hubs` para Azure Event Hubs
- `Azure Service Bus` para Azure Service Bus

### <a name="batch-processing"></a>Procesamiento por lotes
En algunas colas, se pueden quitar de la cola varios mensajes con una solicitud. El procesamiento de este tipo de mensajes es supuestamente independiente y pertenece a las distintas operaciones lógicas. No es posible poner en correlación la operación `Dequeue` con el mensaje determinado que va a procesarse.

Cada mensaje debe procesarse en su propio flujo de control asincrónico. Para más información, vea la sección [Seguimiento de dependencias de salida](#outgoing-dependencies-tracking).

## <a name="long-running-background-tasks"></a>Tareas en segundo plano de ejecución prolongada

Algunas aplicaciones inician operaciones de larga ejecución que es posible que se deban a solicitudes del usuario. Desde la perspectiva del seguimiento o la instrumentación, no es diferente de la instrumentación de solicitudes o dependencias: 

```csharp
async Task BackgroundTask()
{
    var operation = telemetryClient.StartOperation<DependencyTelemetry>(taskName);
    operation.Telemetry.Type = "Background";
    try
    {
        int progress = 0;
        while (progress < 100)
        {
            // Process the task.
            telemetryClient.TrackTrace($"done {progress++}%");
        }
        // Update status code and success as appropriate.
    }
    catch (Exception e)
    {
        telemetryClient.TrackException(e);
        // Update status code and success as appropriate.
        throw;
    }
    finally
    {
        telemetryClient.StopOperation(operation);
    }
}
```

En este ejemplo, `telemetryClient.StartOperation` crea `DependencyTelemetry` y rellena el contexto de correlación. Supongamos que tiene una operación principal creada por las solicitudes de entrada que programaron la operación. Siempre que `BackgroundTask` se inicie en el mismo flujo de control asincrónico que una solicitud de entrada, se correlaciona con esa operación principal. `BackgroundTask` y todos los elementos de telemetría anidados se correlacionan automáticamente con la solicitud que la causó incluso después de que finalice la solicitud.

Cuando la tarea se inicia desde el subproceso en segundo plano que no tiene ninguna operación (`Activity`) asociada a él, `BackgroundTask` no tiene ningún elemento primario. Pero puede tener operaciones anidadas. Todos los elementos de telemetría notificados desde la tarea se ponen en correlación con la `DependencyTelemetry` creada en `BackgroundTask`.

## <a name="outgoing-dependencies-tracking"></a>Seguimiento de dependencias de salida
Puede realizar el seguimiento de su propio tipo de dependencia o de una operación que no es compatible con Application Insights.

El método `Enqueue` de la cola de Service Bus o la cola de Storage puede servir como ejemplo de este seguimiento personalizado.

El enfoque general para el seguimiento de dependencias personalizadas es este:

- Llame al método `TelemetryClient.StartOperation` (extensión) que rellena las propiedades de `DependencyTelemetry` que se necesitan para la correlación y algunas otras (inicio de la marca de tiempo, duración).
- Establezca otras propiedades personalizadas en `DependencyTelemetry`, como el nombre y cualquier otro contexto que se necesite.
- Realice una llamada de dependencia y espere.
- Detenga la operación con `StopOperation` cuando finalice.
- Controle las excepciones.

```csharp
public async Task RunMyTaskAsync()
{
    using (var operation = telemetryClient.StartOperation<DependencyTelemetry>("task 1"))
    {
        try 
        {
            var myTask = await StartMyTaskAsync();
            // Update status code and success as appropriate.
        }
        catch(...) 
        {
            // Update status code and success as appropriate.
        }
    }
}
```

Desechar la operación provoca que la operación se detenga, por lo que puede hacerlo en lugar de llamar a `StopOperation`.

*Advertencia*: en algunos casos, una excepción no controlada puede [impedir](/dotnet/csharp/language-reference/keywords/try-finally) la llamada a `finally`, por lo que puede que no se realice un seguimiento de las operaciones.

### <a name="parallel-operations-processing-and-tracking"></a>Seguimiento y procesamiento de operaciones paralelas

`StopOperation` solo detiene la operación que se inició. Si la operación de ejecución actual no coincide con la que quiere detener, `StopOperation` no hace nada. Es posible que suceda esta situación si se inician varias operaciones en paralelo en el mismo contexto de ejecución:

```csharp
var firstOperation = telemetryClient.StartOperation<DependencyTelemetry>("task 1");
var firstTask = RunMyTaskAsync();

var secondOperation = telemetryClient.StartOperation<DependencyTelemetry>("task 2");
var secondTask = RunMyTaskAsync();

await firstTask;

// FAILURE!!! This will do nothing and will not report telemetry for the first operation
// as currently secondOperation is active.
telemetryClient.StopOperation(firstOperation); 

await secondTask;
```

Asegúrese de llamar siempre a `StartOperation` y de procesar la operación en el mismo método **async** para aislar las operaciones que se ejecutan en paralelo. Si la operación es sincrónica o no asincrónica, encapsule el proceso y realice un seguimiento con `Task.Run`:

```csharp
public void RunMyTask(string name)
{
    using (var operation = telemetryClient.StartOperation<DependencyTelemetry>(name))
    {
        Process();
        // Update status code and success as appropriate.
    }
}

public async Task RunAllTasks()
{
    var task1 = Task.Run(() => RunMyTask("task 1"));
    var task2 = Task.Run(() => RunMyTask("task 2"));
    
    await Task.WhenAll(task1, task2);
}
```

## <a name="applicationinsights-operations-vs-systemdiagnosticsactivity"></a>Operaciones de ApplicationInsights frente a System.Diagnostics.Activity
`System.Diagnostics.Activity` representa el contexto de seguimiento distribuido y lo utilizan las plataformas y las bibliotecas para crear y propagar el contexto dentro y fuera del proceso, así como correlacionar los elementos de telemetría. La actividad funciona junto `System.Diagnostics.DiagnosticSource`, que es el mecanismo de notificación entre la plataforma o la biblioteca para notificar de eventos interesantes (solicitudes entrantes o salientes, excepciones, etc.).

Las actividades son los ciudadanos de primera clase en Application Insights y la recopilación automática de dependencias y solicitudes se basa en gran medida en ellas junto con los eventos `DiagnosticSource`. Si crea una actividad en la aplicación, no se producirá la creación de telemetría de Application Insights. Application Insights debe recibir los eventos de DiagnosticSource y conocer los nombres y las cargas de los eventos para convertir la actividad en telemetría.

Cada operación de Application Insights (solicitud o dependencia) implica `Activity`; cuando se llama a `StartOperation`, se crea la actividad subyacente. `StartOperation` es el método recomendado para realizar el seguimiento de las telemetrías de solicitudes o de dependencias manualmente y asegurarse de que todo está correlacionado.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga los conceptos básicos de la [correlación de telemetría](correlation.md) en Application Insights.
- Consulte cómo los datos correlacionados alimentan la [experiencia de diagnósticos de transacción](./transaction-diagnostics.md) y el [mapa de aplicación](./app-map.md).
- Vea el [modelo de datos](./data-model.md) para los tipos y el modelo de datos de Application Insights.
- Notifique [eventos y métricas](./api-custom-events-metrics.md) personalizados a Application Insights.
- Compruebe la [configuración](configuration-with-applicationinsights-config.md#telemetry-initializers-aspnet) de la recopilación de propiedades de contexto.
- Consulte el [manual del usuario de System.Diagnostics.Activity](https://github.com/dotnet/runtime/blob/master/src/libraries/System.Diagnostics.DiagnosticSource/src/ActivityUserGuide.md) para ver cómo se correlaciona la telemetría.

