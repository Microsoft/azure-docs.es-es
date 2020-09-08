---
title: 'Guía de programación de .NET: Azure Event Hubs (heredado) | Microsoft Docs'
description: En este artículo se proporciona información sobre cómo escribir código para Azure Event Hubs con el SDK de Azure .NET.
ms.topic: article
ms.date: 06/23/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 17bec931f79a6dbb3d98270ab0ff6e2d1d4c6541
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89013918"
---
# <a name="net-programming-guide-for-azure-event-hubs-legacy-microsoftazureeventhubs-package"></a>Guía de programación de .NET para Azure Event Hubs (paquete heredado Microsoft.Azure.EventHubs)
En este artículo se describen algunos escenarios comunes para escribir código mediante Azure Event Hubs. En él se presupone un conocimiento previo de Event Hubs. Para obtener una visión general conceptual de Event Hubs, consulte la [Información general de Azure Event Hubs](./event-hubs-about.md).

> [!WARNING]
> Esta guía se refiere al paquete **Microsoft.Azure.EventHubs** anterior. Se recomienda [migrar](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/eventhub/Azure.Messaging.EventHubs/MigrationGuide.md) el código a fin de usar el paquete [Azure.Messaging.EventHubs](event-hubs-dotnet-standard-getstarted-send.md) más reciente.  


## <a name="event-publishers"></a>Publicadores de eventos

Los eventos se envían a un centro de eventos mediante HTTP POST o una conexión AMQP 1.0. La elección de cuál usar y cuándo depende del escenario específico abordado. Las conexiones AMQP 1.0 se miden como conexiones asincrónicas en Service Bus y son más apropiadas en los escenarios con volúmenes mayores de mensajes frecuentes y requisitos de latencia menores, ya que proporcionan un canal de mensajería persistente.

Si se usan las API administradas de .NET, las construcciones principales para publicar datos en Event Hubs son las clases [EventHubClient][] y [EventData][]. [EventHubClient][] proporciona el canal de comunicación AMQP a través del que se envían eventos al centro de eventos. La clase [EventData][] representa un evento y se usa para publicar mensajes en un centro de eventos. Esta clase incluye el cuerpo, algunos metadatos (Properties) e información de encabezado (SystemProperties) sobre el evento. Otras propiedades se agregan al objeto [EventData][] cuando este atraviesa un centro de eventos.

## <a name="get-started"></a>Introducción
Las clases .NET que admiten Event Hubs se proporcionan en el paquete NuGet [Microsoft.Azure.EventHubs](https://www.nuget.org/packages/Microsoft.Azure.EventHubs/). Puede instalarlas mediante el Explorador de soluciones de Visual Studio o la [Consola del Administrador de paquetes](https://docs.nuget.org/docs/start-here/using-the-package-manager-console) de Visual Studio. Para ello, emita el siguiente comando en la ventana [Consola del administrador de paquetes](https://docs.nuget.org/docs/start-here/using-the-package-manager-console) :

```shell
Install-Package Microsoft.Azure.EventHubs
```

## <a name="create-an-event-hub"></a>Creación de un centro de eventos

Puede usar Azure Portal, Azure PowerShell o la CLI de Azure para crear centros de eventos. Para más información, consulte [Creación de un espacio de nombres de Event Hubs y un centro de eventos con Azure Portal](event-hubs-create.md).

## <a name="create-an-event-hubs-client"></a>Creación de un cliente de Event Hubs

La clase principal para interactuar con Event Hubs es [Microsoft.Azure.EventHubs.EventHubClient][EventHubClient]. Puede crear una instancia de esta clase con el método [CreateFromConnectionString](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.createfromconnectionstring), tal como se muestra en el ejemplo siguiente:

```csharp
private const string EventHubConnectionString = "Event Hubs namespace connection string";
private const string EventHubName = "event hub name";

var connectionStringBuilder = new EventHubsConnectionStringBuilder(EventHubConnectionString)
{
    EntityPath = EventHubName

};
eventHubClient = EventHubClient.CreateFromConnectionString(connectionStringBuilder.ToString());
```

## <a name="send-events-to-an-event-hub"></a>Envío eventos a un centro de eventos

Para enviar eventos a un centro de eventos, cree una instancia [EventHubClient][] y envíela de forma asincrónica mediante el método [SendAsync](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.sendasync). Este método toma un único parámetro de instancia [EventData][] y lo envía de forma asincrónica a un centro de eventos.

## <a name="event-serialization"></a>Serialización de eventos

La clase [EventData][] tiene [dos constructores sobrecargados](/dotnet/api/microsoft.azure.eventhubs.eventdata.-ctor) que aceptan diversos parámetros, bytes o una matriz de bytes, que representan la carga de datos de eventos. Cuando se usa JSON con [EventData][], puede usar **Encoding.UTF8.GetBytes()** para recuperar la matriz de bytes de una cadena codificada con JSON. Por ejemplo:

```csharp
for (var i = 0; i < numMessagesToSend; i++)
{
    var message = $"Message {i}";
    Console.WriteLine($"Sending message: {message}");
    await eventHubClient.SendAsync(new EventData(Encoding.UTF8.GetBytes(message)));
}
```

## <a name="partition-key"></a>Clave de partición

> [!NOTE]
> Si no está familiarizado con las particiones, consulte [este artículo](event-hubs-features.md#partitions). 

Al enviar datos de eventos, puede especificar un valor con hash para generar una asignación de partición. La partición se especifica mediante la propiedad [PartitionSender.PartitionID](/dotnet/api/microsoft.azure.eventhubs.partitionsender.partitionid). Sin embargo, la decisión de usar particiones implica la elección entre disponibilidad y coherencia. 

### <a name="availability-considerations"></a>Consideraciones sobre disponibilidad

El uso de una clave de partición es opcional y debe considerar detenidamente si desea o no utilizar uno. Si no especifica una clave de partición cuando se publica un evento, se usa una asignación de tipo round robin. En muchos casos, el uso de una clave de partición es una buena elección si el orden de los eventos es importante. Cuando se usa una clave de partición, estas particiones requieren la disponibilidad en un solo nodo y las interrupciones pueden producirse a lo largo del tiempo; por ejemplo, cuando los nodos de ejecución se reinician y revisan. Por lo tanto, si establece un identificador de partición y esa partición deja de estar disponible por alguna razón, un intento de acceder a los datos de esa partición provocará un error. Si la alta disponibilidad es más importante, no especifique una clave de partición; en ese caso, los eventos se envían a las particiones que usan el modelo round robin descrito anteriormente. En este escenario, se realiza una selección explícita entre disponibilidad (ningún identificador de partición) y coherencia (anclar eventos a un identificador de partición).

Otra consideración es el control de retrasos en el procesamiento de eventos. En algunos casos podría ser mejor quitar datos y reintentarlo que intentar seguir con el procesamiento, lo que puede provocar más retrasos de procesamiento descendente. Por ejemplo, con un tablero de cotizaciones es preferible esperar a tener todos los datos actualizados, pero en un escenario de chat en directo o VOIP sería preferible disponer de los datos rápidamente, aunque no estén completos.

Dadas estas consideraciones de disponibilidad, en estos casos puede elegir una de las siguientes estrategias de control de errores:

- Detener (dejar de leer Event Hubs hasta que se solucionen cosas)
- Quitar (los mensajes no son importantes, quítelos)
- Reintentar (reintentar los mensajes cuando lo vea apropiado)

Para obtener más información y una explicación sobre las ventajas y desventajas entre la disponibilidad y la coherencia, consulte [Disponibilidad y coherencia en Event Hubs](event-hubs-availability-and-consistency.md). 

## <a name="batch-event-send-operations"></a>Operaciones de envío de eventos por lotes

El envío de eventos por lotes puede ayudar a aumentar el rendimiento. Puede usar la API [CreateBatch](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.createbatch) para crear un lote al que más tarde se pueden agregar objetos de datos para una llamada [SendAsync](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.sendasync).

Un único lote no debe superar el límite de 1 MB de un evento. Además, cada mensaje del lote usa la misma identidad del publicador. Es responsabilidad del remitente asegurarse de que el lote no supera el tamaño máximo del evento. Si es así, se generará un error de **envío** de cliente. Puede utilizar el método auxiliar [EventHubClient.CreateBatch](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.createbatch) para asegurarse de que el lote no supera los 1 MB. Obtiene una salida vacía [EventDataBatch](/dotnet/api/microsoft.azure.eventhubs.eventdatabatch) de la API [CreateBatch](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.createbatch) y, después, use [TryAdd](/dotnet/api/microsoft.azure.eventhubs.eventdatabatch.tryadd) para agregar eventos para construir el lote. 

## <a name="send-asynchronously-and-send-at-scale"></a>Enviar de forma asincrónica y enviar a escala

Los eventos se envían a un centro de eventos de forma asincrónica. Al enviar de forma asincrónica aumenta la velocidad a la que un cliente es capaz de enviar eventos. [SendAsync](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.sendasync) devuelve un objeto [Task](/dotnet/api/system.threading.tasks.task?view=netcore-3.1). Puede usar la clase [RetryPolicy](/dotnet/api/microsoft.servicebus.retrypolicy) en el cliente para controlar opciones de reintento del cliente.

## <a name="event-consumers"></a>Consumidores de eventos
La clase [EventProcessorHost][] procesa los datos de Event Hubs. Debe usar esta implementación para generar los lectores de eventos en la plataforma .NET. [EventProcessorHost][] proporciona un entorno de tiempo de ejecución seguro, seguro para subprocesos y de varios procesos para las implementaciones de procesadores de eventos que también ofrecen administración de concesión de puntos de comprobación y particiones.

Para usar la clase [EventProcessorHost][], puede implementar [IEventProcessor](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor). Esta interfaz contiene cuatro métodos:

* [OpenAsync](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.openasync)
* [CloseAsync](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.closeasync)
* [ProcessEventsAsync](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.processeventsasync)
* [ProcessErrorAsync](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.processerrorasync)

Para iniciar el procesamiento de eventos, cree una instancia de [EventProcessorHost][], proporcionando los parámetros adecuados para el centro de eventos. Por ejemplo:

> [!NOTE]
> EventProcessorHost y sus clases relacionadas se ofrecen en el paquete **Microsoft.Azure.EventHubs.Processor**. Agregue el paquete al proyecto de Visual Studio siguiendo las instrucciones de [este artículo](event-hubs-dotnet-framework-getstarted-send.md#add-the-event-hubs-nuget-package) o emitiendo el comando siguiente en la ventana de la [Consola del Administrador de paquetes](https://docs.nuget.org/docs/start-here/using-the-package-manager-console):`Install-Package Microsoft.Azure.EventHubs.Processor`.

```csharp
var eventProcessorHost = new EventProcessorHost(
        EventHubName,
        PartitionReceiver.DefaultConsumerGroupName,
        EventHubConnectionString,
        StorageConnectionString,
        StorageContainerName);
```

Después, llame a [RegisterEventProcessorAsync](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost.registereventprocessorasync) para registrar su implementación de [IEventProcessor](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor) en el entorno de tiempo de ejecución.

```csharp
await eventProcessorHost.RegisterEventProcessorAsync<SimpleEventProcessor>();
```

En este momento, el host intenta adquirir una concesión en cada partición del centro de eventos mediante un algoritmo "expansivo". Estas concesiones duran un período de tiempo determinado y después deben renovarse. A medida que nuevos nodos, instancias de trabajo en este caso, pasan a estar en línea, colocan reservas de concesión y, con el tiempo, la carga cambia entre los nodos a medida que cada una trata de adquirir más concesiones.

![Host del procesador de eventos](./media/event-hubs-programming-guide/IC759863.png)

Con el tiempo, se establece un equilibrio. Esta capacidad dinámica permite la aplicación del escalado automático basado en CPU a los consumidores para escalar vertical y horizontalmente. Debido a que Event Hubs no tiene un concepto directo de recuentos de mensajes, el uso medio de la CPU suele ser el mejor mecanismo para medir el back-end o la escala del consumidor. Si los publicadores empiezan a publicar más eventos de los que los consumidores pueden procesar, el aumento de la CPU en los consumidores puede usarse para producir una escala automática en el recuento de instancias de trabajador.

La clase [EventProcessorHost][] también implementa un mecanismo de puntos de comprobación basado en el almacenamiento de Azure. Este mecanismo almacena el desplazamiento en función de la partición, para que cada consumidor pueda determinar cuál fue el último punto de comprobación del cliente anterior. A medida que las particiones pasan de un nodo a otro a través de concesiones, este es el mecanismo de sincronización que facilita el desplazamiento de cargas.

## <a name="publisher-revocation"></a>Revocación de publicador

Además de las características avanzadas del entorno en tiempo de ejecución del host del procesador de eventos, el servicio Event Hubs permite la [revocación del editor](/rest/api/eventhub/revoke-publisher) con el fin de impedir a editores específicos el envío de eventos a un centro de eventos. Estas características son útiles si se ha visto comprometido el token de un publicador o si una actualización de software está provocando que se comporten incorrectamente. En estas situaciones, es posible bloquear que la identidad del editor, que forma parte de su token de SAS, publique eventos.

> [!NOTE]
> Actualmente, solo la API REST es compatible con esta característica ([revocación del editor](/rest/api/eventhub/revoke-publisher)).

Para más información acerca de la revocación del publicador y cómo realizar envíos a Event Hubs como publicador, consulte el ejemplo de [Event Hubs Large Scale Secure Publishing](https://code.msdn.microsoft.com/Service-Bus-Event-Hub-99ce67ab) (Publicación segura a gran escala de Event Hubs).

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información acerca de los escenarios de Event Hubs, visite estos vínculos:

* [Introducción a la API de Event Hubs](./event-hubs-samples.md)
* [¿Qué es Event Hubs?](./event-hubs-about.md)
* [Disponibilidad y coherencia en Event Hubs](event-hubs-availability-and-consistency.md)
* [Referencia de la API de host del procesador de eventos](/dotnet/api/microsoft.servicebus.messaging.eventprocessorhost)

[NamespaceManager]: /dotnet/api/microsoft.servicebus.namespacemanager
[EventHubClient]: /dotnet/api/microsoft.azure.eventhubs.eventhubclient
[EventData]: /dotnet/api/microsoft.azure.eventhubs.eventdata
[CreateEventHubIfNotExists]: /dotnet/api/microsoft.servicebus.namespacemanager.createeventhubifnotexists
[PartitionKey]: /dotnet/api/microsoft.servicebus.messaging.eventdata#Microsoft_ServiceBus_Messaging_EventData_PartitionKey
[EventProcessorHost]: /dotnet/api/microsoft.azure.eventhubs.processor
